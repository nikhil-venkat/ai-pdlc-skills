# Employee Detail Panel

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.5

## Summary
A "View" action that opens an accessible side panel (Sheet/Drawer) showing an employee's full
detail — and the host for the AI insights feature.

## Motivation
Part 1 requires a detail view per employee; it's also where the Part 2 AI panel lives. It must be
keyboard accessible and degrade for missing fields.

## Requirements
- "View" opens a focus-trapped, Esc-closable panel for any row, including degraded ones.
- Shows name, email, teams, accounts, tracking status/category, and activity; missing fields degrade.
- Keyboard accessible; closes cleanly and returns focus.

## Open questions
- Single-employee fetch via `employee(id)` vs reuse of the row data already in cache.

## Out of scope
- The AI insights UI itself (ai-consent-flow / ai-insights-fetch / ai-content-safety specs).
- Editing employee data.
