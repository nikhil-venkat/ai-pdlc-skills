---
name: ship
description: Ship a feature to mainline. Use as the final workflow step once a feature is built, simplified, reviewed, and tested — run the pre-merge checklist, merge, and bump the spec to Shipped.
---

# Ship

> Key principle: Faster is safer. Small, frequent, well-verified merges beat big-bang releases.
>
> **Role in the workflow — step 6 of 6.** See [docs/SPECS.md](../../../docs/SPECS.md).
> - **Runs after** `/test` (and only after `/review` has approved).
> - **Input:** a feature whose `spec.md` acceptance criteria are met, tests pass, and review is green.
> - **Output:** the change merged to mainline; the feature's `spec.md` **Status** bumped to `Shipped`.

## Overview

Ship is the controlled hand-off from "done on a branch" to "live on mainline." It is deliberately a
checklist, not a judgment call — every box must be green before merge. For this project, mainline is
`origin/main` of the `ai-employee-insights` repo.

## When to use

- A feature slice is implemented, simplified, reviewed, and its tests pass.
- You are ready to merge to mainline and mark the spec `Shipped`.

**When NOT to use:** mid-implementation, with failing tests, or before `/review` has approved.

## Process

### 1. Pre-merge checklist

- [ ] All acceptance criteria in the feature's `spec.md` are met.
- [ ] `npm run build`, `npm test`, and lint all pass.
- [ ] `/review` approved; blocking issues resolved.
- [ ] Browser-verified against the mock server at `:4000`, including the relevant failure paths
      (forced 5xx / 429 / AI timeout / PII via env knobs) — see the `test` skill.
- [ ] No secrets, PII, or raw AI text committed or logged (telemetry payloads checked).
- [ ] `.env` is gitignored; only `.env.example` is tracked.

### 2. Merge

- Open / finalize the PR (paired with its test PR per `/build`).
- Squash-or-merge to mainline (`main`). Keep history readable — one coherent change per feature.
- Push: commits land on `origin/main` (the submission repo). **Note:** while the project is still in
  active local development the user may ask for *local commits only* — confirm before pushing upstream.

### 3. Update the spec

- Bump the feature's `spec.md` **Status** to `Shipped` and **Last updated** to today.

## Verification

- The change is on mainline and `npm run build` / `npm test` are green on that commit.
- The feature works end-to-end in the browser against the chaotic server.
- The spec **Status** reads `Shipped`.

## Rollback

- First-line mitigation for the AI feature is the **kill switch**: flip the `aiInsights` feature flag
  off (no redeploy needed) — see the `feature-flags` spec and `RUNBOOK.md`.
- For other regressions, revert the merge commit and reopen the spec at `In Progress`.

## Post-ship iteration

When a shipped feature meets real data and an edge case surfaces, don't open a new spec — **edit
`spec.md` in place** and add `research/<YYYY-MM-DD>-<topic>.md`. Status may flip back to `In Progress`
until the follow-up ships. Full rules in [docs/SPECS.md](../../../docs/SPECS.md).
