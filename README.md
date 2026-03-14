# Boiler Plate

A **backup template** and **boilerplate** repository for technology stacks, tooling configs, and project scaffolding. Use this repo as a single source of truth for reusable setups so you can spin up new projects quickly and keep conventions consistent.

---

## Purpose

- **Backup template** — Store and version your preferred stack and config so they are never lost.
- **Boiler plate** — Copy the relevant stack or config into new projects instead of redoing setup each time.
- **Future reuse** — Reduce setup time and keep coding standards, lint rules, and tooling aligned across projects.

---

## Repository structure

| Path | Description |
|------|-------------|
| `nest-js/` | NestJS stack: configs and shared setup (e.g. Biome) for NestJS projects. |
| `nest-js/configs/` | Shared config files (e.g. `biome.jsonc` for lint and format). |
| `react-vite-ts/` | React + Vite + TypeScript stack: configs and shared setup for frontend projects. |
| `react-vite-ts/configs/` | Shared config files (e.g. `biome.jsonc` for lint, format, and React/JSX). |
| `LICENSE` | Project license. |

---

## How to use

1. **Clone or fork** this repo when starting a new project.
2. **Copy** the stack folder you need (e.g. `nest-js/`, `react-vite-ts/`) into your new project, or reference this repo as a template.
3. **Adjust** configs (e.g. Biome, ESLint, TSConfig) per project if needed, then keep this repo updated when you standardize on new settings.

---

## Stacks and configs

### NestJS

- **Configs:** Biome (formatting, linting, organize imports) in `nest-js/configs/biome.jsonc`.
- **Use case:** Node.js/TypeScript backends; copy `configs/` and wire Biome into your NestJS app.

### React + Vite + TypeScript

- **Configs:** Biome in `react-vite-ts/configs/biome.jsonc` (formatting, linting, organize imports, JSX/React rules, sorted classes for `clsx`/`cva`/`tw`).
- **Use case:** React frontends with Vite and TypeScript; copy `configs/` and wire Biome into your React-Vite app.

---

## Contributing to this repo

When you add a new stack or change a config:

- Add or update the matching folder under the repo root (e.g. `nest-js/`, `react-vite-ts/`, or new `next-js/`, etc.).
- Keep README and this table in sync so the repo stays easy to navigate and reuse.

---

## License

See [LICENSE](LICENSE) in this repository.
