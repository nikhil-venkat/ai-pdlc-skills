# Feature Flags

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 0.3

## Summary
A runtime feature-flag mechanism to enable/disable the AI insights feature without redeploying.

## Motivation
Part 3 requires toggling AI insights without a deploy, and the RUNBOOK relies on this as the
first-line kill switch. Operators and reviewers must be able to flip it instantly.

## Requirements
- A `featureFlags` provider exposing an `aiInsights` flag.
- Resolution precedence: **URL param > localStorage > default**.
- `?ff_aiInsights=off` hides the AI feature entirely; the localStorage value persists across reloads.
- Precedence resolution is unit-tested.

## Open questions
- Simple URL/localStorage source vs a polled config endpoint (PLAN open question — leaning simple
  for the take-home; document the tradeoff in DECISIONS.md).

## Out of scope
- A server-driven flag service.
- Per-user targeting or A/B experiments.
