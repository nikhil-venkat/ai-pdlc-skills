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
checklist, not a judgment call — every box must be green before merge. Mainline is `origin/main`
of the `<project-name>` repo.

## When to use

- A feature slice is implemented, simplified, reviewed, and its tests pass.
- You are ready to merge to mainline and mark the spec `Shipped`.

**When NOT to use:** mid-implementation, with failing tests, or before `/review` has approved.

## Process

### 1. Pre-merge checklist

- [ ] All acceptance criteria in the feature's `spec.md` are met.
- [ ] `npm run build`, `npm run test:run`, and `npm run lint` all exit 0.
- [ ] A **verification comment** is attached to the PR with the build/test/lint output
      (see the `build` skill Verification section for the exact format).
- [ ] `/review` approved; all blocking issues resolved.
- [ ] For features with UI: browser-verified against the project's mock/dev server, including
      relevant failure paths (forced 5xx / 429 / timeout / malformed data via env knobs) — see the
      `test` skill. For pure-logic / infrastructure features (telemetry, flags, etc.) this
      item is N/A.
- [ ] No secrets, PII, or other sensitive data committed or logged (telemetry/log payloads checked).
- [ ] `.env` is gitignored; only `.env.example` is tracked.

### 2. Merge via pull request (never merge directly to mainline)

- **Always ship from a feature branch via a PR.** Never commit, push, or merge directly to `main` —
  mainline only ever advances through a reviewed pull request.
- Open / finalize the PR off the feature branch, targeting `main` (paired with its test PR per `/build`).
- Write a standalone PR description: what changed and why, and a link to the feature's `spec.md`.
- **Do not merge the PR or push to `origin/main` yourself.** Leave the merge as an explicit, separate
  step — the PR is merged (squash-or-merge, one coherent change per feature) after review approval,
  by the human or on their explicit say-so. Confirm before merging.

### 3. Bump the spec to Shipped (via a PR — never direct to main)

- **Once the PR is merged**, create a small branch (e.g. `chore/<feature>-shipped`) off the
  freshly-updated `main`, change the feature's `spec.md` **Status** from `In Progress` to
  `Shipped` and **Last updated** to today, commit, push, and open a PR targeting `main`.
- Keep it to one line — the spec bump is its own tiny PR precisely because we never commit
  directly to `main`. The human merges it like any other PR.
- `Shipped` means the code is live on mainline. Don't bump the status until the feature PR
  itself is actually merged.

### 4. Post-merge cleanup

After the human confirms the feature PR has merged:

```bash
# Sync local main to origin/main
git fetch origin --prune
git checkout main
git merge --ff-only origin/main

# Delete the merged branch — local and remote
git branch -d <branch-name>
git push origin --delete <branch-name>
```

`git fetch --prune` removes remote-tracking refs for branches deleted on origin (GitHub may
or may not auto-delete on merge — always run the explicit remote delete to be sure). Deleting
merged branches keeps the branch list clean and signals that work is complete.

## Verification

- The change is on mainline and `npm run build` / `npm run test:run` are green on that commit.
- The feature works end-to-end in the browser against the mock/dev server (UI features only).
- The spec **Status** reads `Shipped`.
- No stale merged branches remain (local or remote).

## Rollback

- First-line mitigation for a flagged feature is the **kill switch**: flip its feature flag
  (e.g. `<feature-flag>`) off (no redeploy needed) — see the project's feature-flag spec and runbook.
- For other regressions, revert the merge commit and reopen the spec at `In Progress`.

## Post-ship iteration

When a shipped feature meets real data and an edge case surfaces, don't open a new spec — **edit
`spec.md` in place** and add `research/<YYYY-MM-DD>-<topic>.md`. Status may flip back to `In Progress`
until the follow-up ships. Full rules in [docs/SPECS.md](../../../docs/SPECS.md).
