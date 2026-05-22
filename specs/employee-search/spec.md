# Employee Search

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.3
**Phase:** 1 — Dashboard

## Summary
A debounced name-search box over the `employees` query that resets pagination and shows a clear
no-results state.

## Motivation
Users need to find a person quickly. Debouncing protects the 60/min GraphQL rate limit by not
firing a request per keystroke.

## Requirements
- Search input with ~300 ms debounce; resets the cursor on change.
- "No results" state when empty; clearable.
- Verified to issue one request per settle, not per keystroke (Network panel + unit test on the
  debounce hook).

## Open questions
- Whether search should also match uid/email (the API does) or name only in the UI copy.

## Out of scope
- Fuzzy/typo-tolerant search; highlighting matches.
