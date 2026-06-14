# Boiler Plate

A **backup template** and **boilerplate** repository for technology stacks, Cursor AI rules, agent skills, tooling configs, and project scaffolding. Use this repo as a single source of truth for reusable setups so you can spin up new projects quickly and keep conventions consistent.

---

## Purpose

- **Backup template** — Store and version your preferred stack, AI rules, and config so they are never lost.
- **Boiler plate** — Copy the relevant stack or config into new projects instead of redoing setup each time.
- **Future reuse** — Reduce setup time and keep coding standards, lint rules, architecture patterns, and AI workflows aligned across projects.

---

## Repository structure

| Path | Description |
|------|-------------|
| `common/` | Shared Cursor rules and Agent Skills used across all stacks. |
| `common/.cursor/rules/` | Cross-stack rules: AI behaviour, code review, unit test, spec generator (PlantUML). |
| `common/.cursor/skills/engineer/` | Reusable Agent Skills: Romano Method, Merlin SA Design, Pirates Codes, Strict Build, Preflight Checklist, KnightGuard/RoyalGuard reviewers, Blacksmith Create Skill, Find Docs. |
| `nest-js/` | NestJS stack: Biome config, Cursor rules, and feature specs. |
| `nest-js/configs/` | Shared config files (e.g. `biome.jsonc` for lint and format). |
| `nest-js/.cursor/rules/` | NestJS-specific rules: governance, tech stack, project structure, security, response control, sensitive data. |
| `nest-js/docs/specs/` | Architecture and feature specs (API context, cache, configuration, logger, sanitize-response). |
| `react-vite-ts/` | React + Vite + TypeScript stack: Biome config and Cursor rules for frontend projects. |
| `react-vite-ts/configs/` | Shared config files (e.g. `biome.jsonc` with JSX/React rules). |
| `react-vite-ts/.cursor/rules/` | Frontend rules: tech stack, performance protocol, unit test. |
| `next/` | Next.js architecture rules (Feature-Sliced Design and Atomic Design). |
| `next/rules/` | Architecture rule files: `architecture-fsd.mdc`, `architecture-atom.mdc`. |
| `LICENSE` | Project license. |

---

## How to use

1. **Clone or fork** this repo when starting a new project.
2. **Copy** the stack folder you need (e.g. `nest-js/`, `react-vite-ts/`, `next/`) into your new project.
3. **Copy** `common/` (or the relevant `.cursor/rules/` and `.cursor/skills/` subsets) to inherit shared AI behaviour and engineering skills.
4. **Wire** configs (e.g. Biome) into your project and link Cursor rules to your workspace.
5. **Adjust** per project as needed, then push improvements back here when you standardize on new settings.

---

## Stacks and configs

### Common (shared across stacks)

- **Cursor rules:** AI behaviour workflow, code review protocol, unit test standards, PlantUML spec generator.
- **Agent Skills:** Debugging (Romano Method), SA design and planning (Merlin), TDD workflow (Pirates Codes / Strict Build), code review (KnightGuard / RoyalGuard), skill authoring (Blacksmith), documentation lookup (Find Docs).
- **Use case:** Copy into any project to give Cursor consistent engineering behaviour regardless of stack.

### NestJS

- **Configs:** Biome (formatting, linting, organize imports) in `nest-js/configs/biome.jsonc`.
- **Cursor rules:** Governance, tech stack, project structure, security, response control (Space Programming), sensitive data handling.
- **Specs:** Feature-level architecture docs under `nest-js/docs/specs/` — API context (CLS), cache module, configuration module, logger, sanitize-response.
- **Use case:** Node.js/TypeScript backends; copy `configs/`, `.cursor/rules/`, and relevant `docs/specs/` into your NestJS app.

### React + Vite + TypeScript

- **Configs:** Biome in `react-vite-ts/configs/biome.jsonc` (formatting, linting, organize imports, JSX/React rules, sorted classes for `clsx`/`cva`/`tw`).
- **Cursor rules:** Tech stack and conventions, performance protocol, unit test standards.
- **Use case:** React frontends with Vite and TypeScript; copy `configs/` and `.cursor/rules/` into your React-Vite app.

### Next.js

- **Architecture rules:** Feature-Sliced Design (`architecture-fsd.mdc`) and Atomic Design (`architecture-atom.mdc`) in `next/rules/`.
- **Use case:** Next.js App Router projects; copy `next/rules/` into your project's `.cursor/rules/` and pair with stack-specific rules from `react-vite-ts/` or your own tech-stack rule.

---

## Contributing to this repo

When you add a new stack or change a config:

- Add or update the matching folder under the repo root (e.g. `nest-js/`, `react-vite-ts/`, `next/`, or a new stack folder).
- Place shared AI rules and skills under `common/`; keep stack-specific rules inside each stack folder.
- Keep README and the structure table in sync so the repo stays easy to navigate and reuse.

---

## License

See [LICENSE](LICENSE) in this repository.
