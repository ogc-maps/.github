# GitHub Projects + Claude Agent Integration

This guide explains how to connect a GitHub Projects board to the Claude agent workflow so that moving an issue between columns automatically triggers the right agent mode.

## Overview

The Claude agent workflow (`.github/workflows/claude.yml`) responds to `status:*` labels on issues and PRs. A companion workflow (`project-label-sync.yml`) polls the project board every 5 minutes and applies the correct label when it detects a status change, bridging the kanban board to the agent pipeline.

## Board Columns

Set up your project board with these Status field values:

| Column | Label Applied | Agent Mode | What Happens |
|---|---|---|---|
| Triage | *(none)* | — | New issues land here for human review |
| Clarify | `status:clarify` | clarify | Agent reads the issue, asks questions, proposes acceptance criteria |
| Ready | `status:ready` | plan | Agent posts a detailed implementation plan |
| In Progress | `status:in-progress` | implement | Agent executes the plan, runs tests, opens a PR |
| In Review | `status:in-review` | review | Agent reviews the PR, approves or requests changes |
| Done | *(none)* | — | Human merges the PR and moves it here |

## Setting Up the Label Sync

The `project-label-sync.yml` workflow runs on a 5-minute schedule and can also be triggered manually. It reads the project board via GraphQL, compares each item's Status to its current labels, and applies or removes `status:*` labels as needed.

### Prerequisites

1. **Create a Personal Access Token (classic)** with these scopes: `project:read`, `repo` (for label writes). Store it as a repository secret named `PROJECT_TOKEN`.

2. **Set the project number** in the workflow's `env` block. The default is `2` (matching `https://github.com/orgs/ogc-maps/projects/2`).

3. **Create the status labels** in each repo that uses the Claude workflow (see table below).

### Why a PAT?

The default `GITHUB_TOKEN` can't read organization-level project boards. A PAT with `project:read` scope is required for the GraphQL query that fetches project items and their statuses.

### Alternative: Manual Labels

If you prefer not to set up the sync workflow, you can apply labels manually. The Claude agent responds to labels regardless of how they're applied:

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
