# AI Content Safety

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 2.3

## Summary
Validate AI-generated text **before it reaches the DOM**: detect and redact PII, gate on
confidence, and capture user feedback. The highest-signal safety feature of the exercise.

## Motivation
The model contaminates ~15% of summaries with PII (phone, SSN, email, address, DOB) and returns
low confidence (<0.30) ~10% of the time. Displaying that verbatim would be a privacy incident and
erode trust. We must never render PII and must clearly mark untrustworthy output.

## Requirements
- **PII detection/redaction** for all five injected types (phone, SSN, email, address, DOB):
  redact/mask before render, show a "sensitive content removed" warning, emit `ai_pii_detected`.
  PII must **never** reach the DOM verbatim — unit-tested per type.
- **Confidence gating**: always show the score; `< 0.30` → caveat banner ("low confidence, verify
  independently") and de-emphasize/optionally collapse the body. `0.9` shows normal treatment.
- **Feedback**: 👍/👎 control emits `ai_feedback` telemetry.
- The content validator is covered by unit tests against known PII/confidence fixtures (the primary deliverable here).

## Open questions
- Redact (mask) vs suppress the whole summary when PII is detected.
- Low-confidence threshold (0.30 matches the server's low-confidence band).

## Out of scope
- Detecting PII types the server never injects (best-effort regex, documented as a known limitation).
- Server-side moderation (we only control the client).
