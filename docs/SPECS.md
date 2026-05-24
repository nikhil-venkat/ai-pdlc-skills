# Specs workflow

A spec is a small, repo-checked folder that captures *what* a feature is, *why* we're
building it, and *what we found* while researching how to build it. Specs live alongside the
code so they're easy to find, update, and audit later.

The `/spec <feature-name>` slash command scaffolds a new spec. Research and implementation
happen conversationally afterward, with the spec files serving as ground truth.

## When to use a spec

Use a spec when:

- The feature is non-trivial (multiple files, multiple decisions, more than a couple hours of work).
- The feature touches an **external API or a security-sensitive path** — anything that runs against
  an external or mock server or an untrusted data source. Edge cases in real responses (null or
  missing fields, malformed data, sensitive-data/PII contamination, low confidence, timeouts, 429s)
  are the most common reason work needs a second pass, and the spec is where those findings get
  recorded.
- Requirements are not obvious from the request and need to be aligned before implementation.

Skip the spec for bug fixes, routine refactors, dependency bumps, or anything small enough that
the PR description would say everything worth saying.

## Folder structure

```
specs/
  <feature-name>/
    spec.md                              # what + why (living document)
    research/
      research.md                        # initial research findings
      <YYYY-MM-DD>-<topic>.md            # follow-up notes (post-ship edge cases, etc.)
```

Feature names are kebab-case (e.g. `audit-log-retention`, `bulk-export`).

Specs are **kept forever** — even after a feature ships. They become institutional memory for
why features look the way they do. Status moves through the lifecycle but the folder stays.

## spec.md — what and why

The user-facing description of the feature. It should be readable by someone who has never seen
the code. Keep it short; if a section is empty, leave it empty rather than padding.

### Template

```markdown
# <Feature Name>

**Status:** Draft
**Last updated:** YYYY-MM-DD
**Plan task:** <task id from PLAN.md, if derived from a plan>

## Summary
<1–2 sentences. What is this feature?>

## Motivation
<Why are we building it? What problem does it solve, for whom?>

## Requirements
<User-facing behavior. Acceptance criteria. What does "done" look like?>

## Open questions
<Things to resolve during research or implementation.>

## Out of scope
<What we are explicitly not doing in this iteration.>
```

### Status values

- **Draft** — spec is being written, requirements not yet stable.
- **Researching** — spec is stable enough to research against; `research/research.md` is being filled in.
- **In Progress** — implementation has started.
- **Shipped** — feature is live in production (merged to mainline).

Status is maintained by hand. When you bump it, also bump **Last updated**.

## research/research.md — how, and what we found

Research is where codebase exploration goes: existing patterns, data shapes, edge cases, and the
proposed approach. This is what implementation works against. Cross-reference any relevant notes in
the root `PLAN.md` and the source of any external or mock server the feature depends on.

### Template

```markdown
# Research: <Feature Name>

## Existing patterns / prior art
<What in the codebase is relevant? Reference files with paths (e.g. `src/lib/api.ts`).>

## Data shapes
<API contracts, value formats this feature touches. Cross-reference the relevant API schema or data source.>

## Edge cases
<What we've found in the data or flow that needs handling.>

## Proposed approach
<How we plan to implement, with file references.>

## Open questions for spec
<Things research surfaced that need product decisions before implementation can finish.>
```

If research surfaces a question that changes the spec, **update spec.md** (don't bury the answer
in research).

## Iteration — handling post-ship findings

Specs are **living documents**. The common case: a feature ships, then an edge case in real API
responses shows up and the feature needs another pass. When that happens:

1. **Edit spec.md in place** to reflect current truth — update Requirements or Out of scope as
   needed, bump **Last updated**. Do not maintain a "Revisions" section; git history is the audit trail.
2. **Add a follow-up file** at `research/<YYYY-MM-DD>-<topic>.md` describing what was learned and
   what changed (e.g. `research/2026-05-25-timeout-handling.md`).

Don't rewrite `research/research.md` after the fact — keep it as the original research record. The
dated follow-ups document the evolution.

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
