# GitHub Projects + Claude Agent Integration

This guide explains how to connect a GitHub Projects board to the Claude agent workflow so that moving an issue between columns automatically triggers the right agent mode.

## Overview

The Claude agent workflow (`.github/workflows/claude.yml`) responds to `status:*` labels on issues and PRs. GitHub Projects can be configured to apply these labels when items move between columns, creating a visual kanban that drives the agent pipeline.

## Board Columns

Set up your project board with these columns (Status field values):

| Column | Label Applied | Agent Mode | What Happens |
|---|---|---|---|
| Triage | *(none)* | — | New issues land here for human review |
| Clarify | `status:clarify` | clarify | Agent reads the issue, asks questions, proposes acceptance criteria |
| Ready | `status:ready` | plan | Agent posts a detailed implementation plan |
| In Progress | `status:in-progress` | implement | Agent executes the plan, runs tests, opens a PR |
| In Review | `status:in-review` | review | Agent reviews the PR, approves or requests changes |
| Done | *(none)* | — | Human merges the PR and moves it here |

## Setting Up Automation

GitHub Projects supports built-in automations and custom workflows. To sync columns with labels:

### Option 1: GitHub Actions Workflow (Recommended)

Create a workflow that watches for project card movements and applies labels:

```yaml
# .github/workflows/project-label-sync.yml
name: Sync Project Status to Labels

on:
  projects_v2_item:
    types: [edited]

jobs:
  sync-labels:
    runs-on: ubuntu-latest
    if: github.event.changes.field_value.field_name == 'Status'
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const statusToLabel = {
              'Clarify': 'status:clarify',
              'Ready': 'status:ready',
              'In Progress': 'status:in-progress',
              'In Review': 'status:in-review',
            };

            const allStatusLabels = Object.values(statusToLabel);
            const newStatus = context.payload.changes.field_value.value?.name;
            const newLabel = statusToLabel[newStatus];

            // Get the issue/PR number from the project item
            const item = context.payload.projects_v2_item;
            const contentId = item.content_node_id;

            // Fetch the issue/PR details via GraphQL
            const query = `query($id: ID!) {
              node(id: $id) {
                ... on Issue { number, repository { owner { login }, name } }
                ... on PullRequest { number, repository { owner { login }, name } }
              }
            }`;
            const result = await github.graphql(query, { id: contentId });
            const node = result.node;
            if (!node?.number) return;

            const owner = node.repository.owner.login;
            const repo = node.repository.name;
            const issue_number = node.number;

            // Remove all existing status labels
            const { data: currentLabels } = await github.rest.issues.listLabelsOnIssue({
              owner, repo, issue_number
            });
            for (const label of currentLabels) {
              if (allStatusLabels.includes(label.name)) {
                await github.rest.issues.removeLabel({
                  owner, repo, issue_number, name: label.name
                });
              }
            }

            // Apply the new label (if the column maps to one)
            if (newLabel) {
              await github.rest.issues.addLabels({
                owner, repo, issue_number, labels: [newLabel]
              });
            }
```

### Option 2: Manual Label Application

If you prefer simplicity, skip the automation and apply labels manually. The agent responds to labels regardless of how they're applied:

1. Move an issue to a column on the board.
2. Apply the matching `status:*` label on the issue.
3. The Claude workflow triggers automatically.

## Required Labels

Create these labels in each repo that uses the Claude workflow:

| Label | Color (suggested) | Description |
|---|---|---|
| `status:clarify` | `#d4c5f9` | Agent: ask clarifying questions |
| `status:ready` | `#0e8a16` | Agent: create implementation plan |
| `status:in-progress` | `#fbca04` | Agent: implement and open PR |
| `status:in-review` | `#1d76db` | Agent: review PR |

## Typical Workflow

1. Create an issue describing the task (use the "Agent task" template if available).
2. Add it to the project board — it lands in **Triage**.
3. Move to **Clarify** (or apply `status:clarify`) — agent asks questions.
4. Answer the agent's questions in comments.
5. Move to **Ready** (or apply `status:ready`) — agent posts a plan.
6. Review the plan. Move to **In Progress** (or apply `status:in-progress`) — agent implements and opens a PR.
7. Move the PR to **In Review** (or apply `status:in-review`) — agent reviews the code.
8. Human merges the PR. Move to **Done**.
