# Repository Guidelines

## Project Structure & Module Organization

GenChemPrepHW is being designed as a hosted homework platform for college-level general chemistry. Current planning artifacts live in `docs/superpowers/specs/`. As implementation begins, use:

- `src/` for application code.
- `tests/` for automated tests mirroring source modules.
- `assets/` for static files, fixtures, and source samples.
- `docs/` for design notes, specs, and architecture decisions.

Keep grading, validation, content ingestion, and assignment workflows in clearly separated modules.

## Build, Test, and Development Commands

No application stack has been scaffolded yet. Once the project is initialized, document the exact commands here. Expected defaults for the planned TypeScript web app are:

- `npm install` to install dependencies.
- `npm run dev` to run the local development server.
- `npm test` to run automated tests.
- `npm run build` to create a production build.

Run commands from the repository root.

## Coding Style & Naming Conventions

Use TypeScript for application code unless the architecture changes. Prefer strict types, small modules, and deterministic grading functions. Use `PascalCase` for React components, `camelCase` for variables/functions, and `kebab-case` for route or asset filenames.

Add formatting and linting early, preferably Prettier and ESLint.

## Testing Guidelines

Accuracy is critical. Tests must cover grading rules, numeric tolerances, units, significant figures, answer normalization, assignment submissions, and role permissions. Name tests after the behavior under test, such as `grading-units.test.ts`.

Every fixed chemistry, grading, or validation bug should receive a regression test.

## Commit & Pull Request Guidelines

Use concise imperative commit messages, for example `Add question validation spec`. Pull requests should include a summary, test results, linked issues, and screenshots for UI changes.

## Agent-Specific Instructions

Do not expose generated questions to graded workflows unless they are validated and instructor-approved. Preserve audit trails, source alignment, and question version history.
