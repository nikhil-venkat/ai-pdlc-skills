# Specs & the Development Workflow

> **Source of truth** for how a feature moves from idea to mainline in `<project-name>`.
>
> The `/spec`, `/build`, `/code-simplify`, `/review`, `/test`, and `/ship` skills all defer to this
> document for the workflow, folder structure, templates, status values, and iteration rules. When a
> skill and this doc disagree, **this doc wins** — read it first.

## Philosophy

**Spec before code.** Every non-trivial change starts as a short written spec so that intent is
agreed before implementation. The spec is a *living document*: it is created up front, kept accurate
while the feature is built, and updated in place when reality teaches us something after shipping.

Three properties keep the system honest:

- **One spec per feature**, in its own folder, so work is self-contained and reviewable.
- **Status is explicit** (`Draft` → `In Progress` → `Shipped`) so anyone can see where a feature
  stands at a glance.
- **Git history is the audit trail.** We edit specs in place and add dated follow-ups rather than
  rewriting history — the diff log tells the story of how the feature evolved.

## The workflow (6 steps)

Each feature flows through six steps, each backed by a skill. Steps run in order; later steps may
send you back to an earlier one (a failed review sends you back to build).

| # | Skill            | Purpose                                                        | Status after        |
|---|------------------|----------------------------------------------------------------|---------------------|
| 1 | `/spec`          | Define **what** to build and **why**, before any code.         | `Draft`             |
| 2 | `/build`         | Implement the feature in small, working slices + tests.        | `In Progress`       |
| 3 | `/code-simplify` | Polish the new code for clarity; behavior unchanged.           | `In Progress`       |
| 4 | `/review`        | Multi-axis review against the spec's acceptance criteria.      | `In Progress`       |
| 5 | `/test`          | Prove it works — automated tests + runtime/browser checks.     | `In Progress`       |
| 6 | `/ship`          | Merge to mainline via PR; record the feature as done.          | `Shipped`           |

```
/spec → /build → /code-simplify → /review → /test → /ship
  │        ▲           ▲              │                  │
  │        └───────────┴──────────────┘ (review/test     │
  └─ Draft                              can loop back)    └─ Shipped
```

Branching and PR discipline (shared by every step that writes code):

- **Create a feature branch off mainline (`main`) before making any changes.** Never commit directly
  to mainline.
- **Land every change as a reviewed PR** targeting `main`. Pair each code-change PR with a test PR
  (per `/test`).
- **Attach a verification comment to each PR** with the build/test/lint output, so reviewers and
  future readers never have to re-run the checks to know they passed (see the `/build` skill for the
  exact format).

## Folder structure

Every feature gets one folder under `specs/`, named in kebab-case:

```
specs/
  <feature-name>/
    spec.md                       # What & why — a living document (edited in place)
    research/
      research.md                 # Initial investigation — written once, not rewritten
      <YYYY-MM-DD>-<topic>.md      # Post-ship follow-up findings (one file per finding)
```

- **Kebab-case the feature name** (e.g. "Audit log retention" → `audit-log-retention`).
- `spec.md` is mandatory. `research/research.md` is created alongside it and holds the up-front
  investigation.
- Dated `research/<YYYY-MM-DD>-<topic>.md` files accumulate over time as the feature meets real data.

## Status values

`spec.md` carries a **Status** field with exactly one of these values:

| Status        | Meaning                                                        | Set by    |
|---------------|----------------------------------------------------------------|-----------|
| `Draft`       | Spec is being written; implementation has not started.         | `/spec`   |
| `In Progress` | Implementation is underway (build / simplify / review / test). | `/build`  |
| `Shipped`     | The change is merged and live on mainline.                     | `/ship`   |

```
Draft ──/build──▶ In Progress ──/ship──▶ Shipped
                       ▲                     │
                       └──── post-ship ──────┘
                         finding reopens it
```

A `Shipped` feature may flip back to `In Progress` when a post-ship finding requires a follow-up,
then return to `Shipped` when that follow-up ships.

## `spec.md` template

Copy this when scaffolding a new spec. Use today's date for **Last updated** and set **Status** to
`Draft`.

```markdown
# <Feature name>

- **Status:** Draft
- **Last updated:** <YYYY-MM-DD>
- **Owner:** <name or team>

## Summary

One or two sentences: what this feature is, in plain language.

## Motivation

Why build this, and why now? What problem does it solve, and for whom? What happens if we don't?

## Requirements

What the feature must do — functional and non-functional. Be specific enough that someone else could
implement it. Link to designs, APIs, or constraints where relevant.

## Acceptance criteria

Concrete, testable conditions that define "done." `/review`, `/test`, and `/ship` check against this
list — keep it sharp.

- [ ] <observable behavior or invariant the feature must satisfy>
- [ ] <edge case / failure path that must be handled>
- [ ] <non-functional bar: performance, accessibility, security, etc.>

## Open questions

- <unresolved decision blocking or shaping the work>

## Out of scope

- <explicitly excluded so the boundary is clear and the change stays small>
```

## `research/research.md` template

The up-front investigation. Written once during the spec/research phase; **don't rewrite it after the
fact** — later findings go in dated follow-up files instead.

```markdown
# Research: <Feature name>

- **Last updated:** <YYYY-MM-DD>

## Questions to answer

- <what we need to learn before/while building>

## Findings

What the investigation turned up — data, constraints, existing-code behavior, prior art.

## Options considered

| Option | Pros | Cons |
|--------|------|------|
| A      |      |      |
| B      |      |      |

## Decision

Which option we chose and why. Note the trade-offs we accepted.

## References

- <links to docs, issues, benchmarks, design files>
```

## Iteration rules

**While building (Draft → Shipped):** keep `spec.md` accurate as the implementation reveals reality.
Update Requirements, Acceptance criteria, and Open questions in place, and bump **Last updated**.

**After shipping (post-ship findings):** when a shipped feature meets real data and an edge case
surfaces, **do not open a new spec**. Instead:

1. **Edit `spec.md` in place** to reflect the new understanding, and bump **Last updated**.
2. **Add a `specs/<feature-name>/research/<YYYY-MM-DD>-<topic>.md`** file describing the finding and
   what you changed.
3. **Never rewrite `research/research.md` after the fact** — it is the record of the original
   investigation. Git history is the audit trail.
4. Flip **Status** back to `In Progress` if a follow-up change is underway; return it to `Shipped`
   once that follow-up ships.

## Conventions summary

- One spec per feature, kebab-cased, under `specs/<feature-name>/`.
- `spec.md` is the living source of truth; `research/research.md` is write-once.
- Status is always one of `Draft`, `In Progress`, `Shipped`.
- Branch off `main`; never commit to mainline directly; every change ships as a reviewed PR with a
  verification comment.
- Edit in place and add dated follow-ups; let git history carry the story.
