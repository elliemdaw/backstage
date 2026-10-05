---
alwaysApply: true
---

Backstage is an open platform for building developer portals. This is a TypeScript monorepo using Yarn workspaces.

## Key Directories

- `/packages`: Core framework packages (prefixed `@backstage/`)
- `/plugins`: Plugin packages (prefixed `@backstage/plugin-*`)
- `/packages/app`: Main example app using the new frontend system
- `/packages/app-legacy`: Example app using the old frontend system
- `/packages/backend`: Example backend for local development
- `/docs`: Documentation files

Packages prefixed with `core-` (e.g., `@backstage/core-plugin-api`) are part of the old frontend system. Packages prefixed with `frontend-` (e.g., `@backstage/frontend-plugin-api`) are part of the new frontend system. Packages prefixed with `backend-` (e.g., `@backstage/backend-plugin-api`) are part of the backend system.

## Writing Standards

Changes to the docs should follow the documentation style guide at `/docs/contribute/doc-style-guide.md`.

## Code Standards

The following files contain guidelines for the project:

- `/CONTRIBUTING.md`: comprehensive contribution guidelines.
- `/STYLE.md`: guidelines for code style.
- `/REVIEWING.md`: guidelines for pull requests and writing changesets.
- `/SECURITY.md`: guidelines for security.
- `/docs/architecture-decisions/`: contains the architecture decisions for the project.

All new source files (`.ts`, `.tsx`, `.js`, `.jsx`) must include an Apache 2.0 copyright header with the current year. This does not apply to generated files, configuration files (JSON, YAML), or documentation files. Do NOT update the copyright year on existing files — leave the original year as-is.

When writing or generating code, always match the existing coding style of each individual package and file. Different packages in the monorepo may have different conventions — consistency within a package is more important than consistency across the repo.

When writing or generating tests, prefer fewer thorough tests with multiple assertions over many small tests. When using React Testing Library, prefer using `screen` and `.findBy*` queries over `waitFor`, and avoid adding test IDs to the implementation.

## Development Flow

Before any of these commands can be run, you need to run `yarn install` in the project root.

- Build: There is no need to build the project during development, and it is verified automatically in the CI pipeline.
- Test: Use `CI=1 yarn test <path>` in the project root to run tests. The path can be either a single file or a directory. Always provide a path, avoid running all tests.
- Type checking: Use `yarn tsc` in the project root to run the type checker. Do not try to run it somewhere else than the project root and do not supply any options.
- Code formatting: Use `yarn prettier --write <...paths>` to format code. Run it explicitly for file paths that you know are changed, not for entire folders - otherwise it may change formatting of unrelated files.
- Lint: Use `yarn lint --fix` in the project root to run the linter.
- API reports: Before submitting a pull request with changes to any package in the workspace, run `yarn build:api-reports` in the project root to generate API reports for all packages.
- Dev server: Use `yarn start` to run the example app locally (frontend on :3000, backend on :7007).
- Create: Use `yarn new` to scaffold new plugins, packages, or modules.

You MUST NOT run builds or create a release by running `yarn build`, `yarn changesets version`, or `yarn release` as part of any changes. Builds and releases are made by separate workflows.

All changes that affect the published version of packages in the `/packages` and `/plugins` directories must be accompanied by a changeset. Changes outside of these directories (e.g. `.patches/`, `.github/`, `docs/`, root config files) do not need changesets. Only non-private packages require changesets. See the guidelines in `/CONTRIBUTING.md#creating-changesets` for information on how to write good changesets. Changesets are stored in the `/.changeset` directory and should be created by writing changeset files directly — never use the changeset CLI. Breaking changes must be accompanied by a `minor` version bump for packages below version `1.0.0`, or a `major` version bump for packages at version `1.0.0` or higher. For non-breaking changes that introduce new APIs or features, use `minor` for packages at version `1.0.0` or higher, and `patch` for packages below `1.0.0`. Each changeset message should be relevant to the specific package it targets and written for Backstage adopters as the audience — describe user-facing behavior changes in plain language. Never reference internal implementation details such as function names, class names, variable names, or other code symbols that are not part of the public API. If a change spans multiple packages you often need to create separate changesets to make sure they are tailored to each package.

Changes that introduce new features or modify existing behavior must include documentation updates. Documentation should be placed in [TSDoc](https://tsdoc.org) comments, the package README, or within the `/docs` folder, whichever is most appropriate. Documentation should follow the style guide at `/docs/contribute/doc-style-guide.md`.

Before creating a pull request, check whether there is already an open PR for the same change to avoid duplicating effort.

When creating pull requests, use the template at `/.github/PULL_REQUEST_TEMPLATE.md`. Do NOT erase or replace the template — fill it in and only check items on the checklist that have actually been completed. PR descriptions should be short and concise. If there are extensive details to share (design rationale, migration context, investigation notes), suggest opening a GitHub issue and linking to it from the PR instead. If the PR is related to an existing issue, link to it in the PR description.

Never update ESLint, Prettier, or TypeScript configuration files unless specifically requested.

Never make changes to the release notes in `/docs/releases` unless explicitly asked. These document past releases and should not be updated based on newer changes.

## Repository Structure

See `/docs/contribute/project-structure.md` for a detailed description of the repository structure.

<!-- gjalla-start -->
## gjalla System-level Version Control

This project uses gjalla for context management, agent orchestration, spec management, and rule enforcement. Project ID: 87

### Setup
Ensure gjalla CLI is installed: `pip install gjalla` (or `pipx install gjalla`), then you can use `gjalla setup` to get going. If there's already a .gjalla dir and a config.yaml, setup may have already been successfully run. You'll know for sure by checking that gjalla is represented in precommit hooks.

### System-level Context                                                                                                                                                                                                               
Gjalla is the persistent architectural memory layer for this codebase. It tracks architecture, rules, capabilities, data flows, tech stack, and any custom context that you need to know to operate on this team — and records how they change over time. Use it to ground your work in the actual system state and the expectations of your team before writing code, and to keep the record accurate after you commit.

Institutional knowledge and helpful context are in .gjalla/guidance if available. The gjalla CLI is the fastest, most reliable way to get context about architecture and behavior. Use these resources during planning and as you implement to understand the intended architecture, rules, and system constraints. The CLI is best for local project context, and the MCP is best if you need gjalla changelogs, findings history, or system-wide context (your whole team works on these and the centralized storage is remote).

Key commands:
- `gjalla state show` — current system state (elements, rules, capabilities, etc.)
- `gjalla rules list` — constraints and ADRs you must follow
- `gjalla log [--since 7d] [--type ...]` — semantic change history for the project
- `gjalla context show` — agent-ready context bundle for this project
- `gjalla spec new <slug>` — scaffold a change spec before implementing

Fetch relevant context via gjalla to orient your design. Prefer doing things the right way — in alignment with the project's architecture, rules, and best practices — over expedient shortcuts. The goal should be clean, organized implementations that are not over-complicated, don't cause collateral effects, and are easy to understand and maintain. When we do this right, our implementations are usually simpler to implement, test, and maintain, since they align with the source of truth.

### Planning, Implementing, and Preparing for Commit                             
Source-of-truth specs can be fetched with gjalla. If your change affects any system primitive (architecture, capabilities, data model, surface area, data flows, external services, rules, or tech stack), create a change spec. gjalla also has helpful skills to help you construct specs, review and harden specs, breakdown your tasks, audit your test system, and other helpful steps as you proceed through the SDLC.

### Committing
To help your team understand and track your changes, you must attest to all of your changes and guardrail adherence via a gjalla attestation. You must review all your changes for adherence to gjalla rules, fixing important violations and flagging any that need human review. You will also be required to report how the staged code changes the system and the provenance of your work. `gjalla attest --example` has all the fields, view the output in full. Focus only on staged code only even if you're worked on other changes in this session.
Your attestation will be externally validated, so ensure you're accurate and complete.

### Windows Note
On Windows, Git for Windows includes its own POSIX shell (MSYS2), so the shell commands above (e.g. `shasum`, `cut`) work inside git hooks and Git Bash. You do not need WSL or a separate Unix environment.

### Available Skills
Run `gjalla skills show <slug>` for the full body of a skill.
- **verify** (`verify`): Verify implementation against change spec, rules, upstream/downstream, and state of the system. Use after completing implementation to ensure correctness and completeness.
- **spec-create** (`spec`): Create a feature specification with problem statement, goals, technical approach, and testing strategy. Use before implementing any non-trivial feature.
- **test-audit** (`test-audit`): A process to identify the effectiveness of tests in the repository, helpful for identifying if any tests are providing false confidence and whether there are hotspots which may be riskier than initially thought. Use when wanting to improve system robustness, audit current tests, or improve coverage.
- **task-breakdown** (`breakdown`): Break feature spec into sized tasks grouped by dependency. Use after a spec is written to plan implementation.
- **manage-tech-debt** (`manage-tech-debt`): Identify and prioritize tech debt using state lifecycle states and rules. Use for periodic debt assessment or before planning sprints.
- **write-runbook** (`write-runbook`): Write operational procedures using state for service topology and external services. Use when documenting ops procedures for a service.
- **spec-review** (`spec-review`): Full review to assess architecture alignment, security posture, quality and robustness, user-impact, and governance. Use to reviewing a feature spec before implementation.
- **implement** (`implement`): Execute task manifest (produced during task breakdown), pausing to verify the waves of effort. Use to implement plans.
- **conduct-post-mortem** (`conduct-post-mortem`): Blameless incident analysis with 5 Whys, reading state for affected elements. Use after production incidents to identify root cause and action items.
<!-- gjalla-end -->
