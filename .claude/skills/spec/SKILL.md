---
name: spec
description: Define what to build before writing code. Use when starting a new feature, when requirements are vague, or before any non-trivial change.
---

# Spec

> Key principle: Spec before code.
>
> **Role in the workflow — step 1 of 6.** See [docs/SPECS.md](../../../docs/SPECS.md) for the
> full workflow, folder structure, templates, status values, and iteration rules. This skill is the
> command behavior; `docs/SPECS.md` is the source of truth — read it first and follow it precisely.

Run this skill to scaffold and drive a spec before a feature is implemented. Each feature-building
task gets its own spec under `specs/<feature-name>/` (`spec.md` + `research/research.md`). Research
and implementation happen conversationally afterward, with the spec files treated as ground truth by
`/build`.

## Behavior of this command

Start or resume work on a spec called "$ARGUMENTS".

1. **Resolve the feature name.** Kebab-case `$ARGUMENTS` (e.g. "Audit log retention" →
   `audit-log-retention`). If `$ARGUMENTS` is empty, ask the user for a name before doing anything else.

2. **Check for an existing spec.** If `specs/<name>/` already exists, do **not** scaffold or overwrite. Instead:
   - Read `specs/<name>/spec.md` and any files under `specs/<name>/research/`.
   - Tell the user the current **Status** and summarize where things stand.
   - Ask what they want to do next (revise the spec, continue research, start implementation, log a
     post-ship finding, etc.) and proceed accordingly.

3. **Otherwise, scaffold.** Create:
   - `specs/<name>/spec.md` from the `spec.md` template in [docs/SPECS.md](../../../docs/SPECS.md).
   - `specs/<name>/research/research.md` from the `research.md` template in the same doc.
   Use today's date for **Last updated** in `spec.md`. Set **Status** to `Draft`.

4. **Drive the spec phase.** After scaffolding, ask focused clarifying questions to populate `spec.md` —
   cover Summary, Motivation, Requirements, Open questions, and Out of scope. Save updates as the
   conversation progresses; don't batch them to the end. Don't ask everything at once.

5. **End the turn.** Once `spec.md` is in reasonable shape (Summary + Motivation + Requirements at
   minimum), report what was created and ask whether to start research now or step away. Don't auto-start research.

## Iteration rule

For follow-ups (not initial scaffolding): when post-ship findings come up, **edit `spec.md` in place**
(bump **Last updated**) and add a `specs/<name>/research/<YYYY-MM-DD>-<topic>.md` follow-up. Never
rewrite `research/research.md` after the fact — git history is the audit trail. Full details in
[docs/SPECS.md](../../../docs/SPECS.md).
