# App Scaffold

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 0.1
**Phase:** 0 — Foundation

## Summary
Scaffold the Vite + React + TypeScript app and dev tooling into the existing repo so the feature
specs have a working foundation to build on.

## Motivation
Every feature spec (table, AI insights, telemetry, …) needs a running app shell, a place to host the
data layer, a test runner, and a styling system. This is the one-time foundation. The submission repo
itself is **already wired** — cloned, `origin` set to `ai-employee-insights`, with `.claude/` skills,
`docs/SPECS.md`, and the `specs/` folder — so this task is the app/tooling scaffold, not repo setup.

## Requirements
- Vite + React + TS app scaffolded **into the existing repo**, preserving `.claude/`, `docs/`,
  `specs/`, and `LICENSE` (the dir is non-empty — scaffold via a temp dir + merge, per PLAN.md 0.1).
- Tailwind + shadcn/ui initialized; a sample primitive (e.g. `button`) renders.
- Vitest + React Testing Library wired; ESLint + Prettier configured.
- `.env` with `VITE_API_BASE=http://localhost:4000`; `.env.example` committed; `.gitignore` covers
  `node_modules/`, `dist/`, and `.env`.
- Account icons + Faros logo copied into `src/assets/` from `../frontend-exercise-template-main/assets/`.
- `npm run dev` serves a blank shell; `npm run build` and `npm test` both pass.

## Open questions
- Package manager: npm (default) vs pnpm.
- Whether to add a minimal CI workflow now or as a later spec (likely later).

## Out of scope
- Any feature implementation — each is its own spec (data-fetching-and-resilience, employee-table, …).
- Repo wiring / remote setup (already done).
