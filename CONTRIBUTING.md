# Contributing to OGC Maps

Thanks for your interest in contributing! These guidelines apply across all repositories in the ogc-maps organization.

## Reporting Bugs

Open a GitHub Issue in the relevant repository. Include:

- Steps to reproduce the problem
- Expected vs. actual behavior
- Browser, OS, and Node.js version
- Screenshots or error logs if applicable

## Suggesting Features

Open a GitHub Issue with the `enhancement` label. Describe the use case and why it would benefit the project. If you have a design in mind, sketches or mockups are welcome.

## Pull Request Process

1. Fork the repository and create a feature branch from `main`.
2. Make your changes. Follow existing code style (TypeScript, functional components, Zod schemas for validation).
3. Run `pnpm verify` (or the repo's equivalent) and ensure it passes before opening your PR.
4. Open a pull request targeting `main`. Link any related issues.
5. A maintainer (or the Claude agent) will review your PR. Address any feedback, then a human will merge it.

### Using the Claude Agent

Several repos include a Claude CI workflow. You can interact with it on issues:

- `@claude clarify` — asks clarifying questions and proposes acceptance criteria
- `@claude plan` — posts a detailed implementation plan
- `@claude implement` — executes the plan and opens a PR
- `@claude review` — reviews a PR for convention compliance and code quality

See `.github/workflows/claude.yml` in each repo for details.

## Code Style

- TypeScript with strict mode
- pnpm for package management
- Prettier and ESLint where configured
- Prefer small, focused PRs over large changesets

## License

All contributions are licensed under the AGPL-3.0, the same license as the project. By submitting a pull request, you agree that your contributions will be licensed under these terms.
