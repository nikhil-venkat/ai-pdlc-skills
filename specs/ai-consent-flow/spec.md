# AI Consent Flow

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 2.1
**Phase:** 2 — AI Insights

## Summary
An explicit, human-in-the-loop consent step that obtains and manages the consent token required
before any AI insight request, gated behind the `aiInsights` feature flag.

## Motivation
The AI endpoint refuses to respond without a consent token (`X-Consent-Token`), and consent is a
deliberate privacy boundary distinct from auth. Users must knowingly opt in to AI processing of
employee data; tokens expire after 1 hour and must be re-obtained gracefully.

## Requirements
- Before any insight call, show consent UI explaining what's shared and why.
- On accept → `POST /api/ai/consent` `{ userId, scope: "insights" }`; store token + `expiresAt`.
- No insight request fires without a valid token; an expired token triggers re-consent (not a raw 403).
- When the `aiInsights` flag is off, the AI section is absent entirely.
- Emit `ai_consent_granted` telemetry on acceptance.

## Open questions
- `userId` value with no auth — use a synthetic session id (e.g. `demo-user`); confirm + document.
- Persist consent for the session only vs across reloads.

## Out of scope
- Real authentication; per-employee consent scoping.
