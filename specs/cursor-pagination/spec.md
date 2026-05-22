# Cursor Pagination

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.2
**Phase:** 1 — Dashboard

## Summary
Relay-style cursor pagination over the `employees` connection, with stable paging that composes
with search and filters.

## Motivation
The API is cursor-paginated (not offset); the table must page through results without flicker and
reset correctly when the query criteria change.

## Requirements
- Next/Prev (or "Load more") driven by `pageInfo.hasNextPage` / `endCursor`.
- `keepPreviousData` so rows don't flash on page change.
- Page-size control and `totalCount` display.
- Controls disabled at boundaries; cursor resets when search/filters change.

## Open questions
- Next/Prev vs infinite "Load more" UX.
- Default page size (10).

## Out of scope
- Server-side sorting.
