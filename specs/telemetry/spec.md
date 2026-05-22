# Telemetry

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 0.3, 3.1

## Summary
Structured telemetry to `POST /api/telemetry`: a non-throwing client plus instrumentation of the
key user and AI events across the app.

## Motivation
Production thinking and the RUNBOOK depend on observability. We need to know consent rates, AI
success/failure/timeout rates, PII detections, and feature usage — without ever leaking PII.

## Requirements
- Telemetry client: batched, flushes via `navigator.sendBeacon` / `fetch` keepalive on unload,
  and **never throws** (a telemetry failure must not affect the UI).
- Event taxonomy: `page_view`, `employee_search`, `employee_filter`, `employee_paginate`,
  `employee_view`, `ai_consent_granted`, `ai_insight_requested`, `ai_insight_succeeded`,
  `ai_insight_failed`, `ai_insight_timeout`, `ai_pii_detected`, `ai_low_confidence`, `ai_feedback`.
- Consistent envelope: `{ event, ts, sessionId, props }`.
- **No PII or raw AI summary text** in any payload (asserted in review).
- Each user action emits exactly one well-formed event (verified at `GET /api/telemetry`).

## Open questions
- Batch size / flush interval.
- `sessionId` generation (random per page load is sufficient — no auth).

## Out of scope
- A real analytics backend or dashboards (the RUNBOOK names the metrics to watch).
