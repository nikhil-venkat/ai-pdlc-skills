# Error Boundaries

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 0.3, 3.2

## Summary
React error boundaries that isolate failures so one broken section (e.g. the AI panel) never
blanks the whole page.

## Motivation
The API is chaotic and AI content is untrusted, so failures must be contained with retry
affordances rather than crashing the app — a Part 3 production-readiness requirement.

## Requirements
- A root boundary plus section-level boundaries (table vs AI panel vs filters).
- A thrown error in the AI panel leaves the table interactive.
- A thrown error in a single row cell is contained, not page-wide.
- Fallback UIs offer a retry where sensible.

## Open questions
- Boundary granularity — per-row vs per-table (cost vs isolation).

## Out of scope
- Error reporting to a third-party service (telemetry handles event capture).
