# Employee Table

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.1

## Summary
The core employee table: fetch from the GraphQL `employees` query and render Name (avatar),
Tracking Status, Teams, and Accounts Connected — degrading gracefully on dirty data.

## Motivation
This is the primary view of Part 1 and the host for every other dashboard feature (search,
filters, pagination, detail panel). It must look professional and never crash on the dataset's
known data-quality landmines.

## Requirements
- Columns: **Name** (avatar + initials fallback on image error), **Tracking Status** (badge),
  **Teams** (chips; "Unassigned" when empty), **Accounts Connected** (type icons; empty state).
- All 32 rows render. Degraded rows (`arthur` null name, `voldemort` missing email/photo/teams/
  accounts) render with **no "null" or blank crash**; name falls back to uid.
- Loading → skeleton; fetch error → inline retry (not a blank page); empty result → empty state.
- Must match the provided Figma once screenshots are available.

## Open questions
- Exact Figma spacing/colors/row height (awaiting screenshots).
- Whether to show `inactive`/`trackingCategory` in the table or reserve for the detail panel.

## Out of scope
- Pagination, search, filters, detail panel (separate specs).
