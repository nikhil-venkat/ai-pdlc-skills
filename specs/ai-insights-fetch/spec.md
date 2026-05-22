# AI Insights Fetch

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 2.2

## Summary
Fetching AI insights from `GET /api/ai/insights/:id` with the consent token, handling the full
range of real-LLM behaviors: variable latency, timeouts, and distinct error states.

## Motivation
The endpoint behaves like a real model service — 200 ms–5 s latency, ~5% timeouts (504 after
10–20 s), a 10/min rate limit, and several error codes. The UX must stay responsive and never
hang, degrading gracefully on every failure mode.

## Requirements
- Request with `X-Consent-Token`; loading state for up to 5 s (skeleton + "generating", not a frozen spinner).
- **Client `AbortController` timeout (~8–10 s)** → friendly timeout fallback + manual "Try again"
  (respecting the 10/min limit and surfacing `Retry-After`).
- Distinct, non-crashing messages for 401 / 403 / 404 / 429 / 504.
- Normal response renders summary + metadata (model, `processingTimeMs`, `generatedAt`).
- Per-employee in-session caching to avoid tripping the rate limit, with a manual "regenerate".

## Open questions
- Exact client timeout threshold (8 s vs 10 s) vs the server's 10–20 s timeout window.

## Out of scope
- Content safety / PII redaction / confidence gating (ai-content-safety spec).
- Streaming responses (endpoint returns a single JSON payload).
