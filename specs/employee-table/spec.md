# Employee Table

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.1
**Design reference:** [Figma — Employees Page](https://www.figma.com/file/nR9Voet7KCWpOYNJzB4O4Z/Frontend-Engineer-Interview-Exercise%3A-Employees-Page?type=design&node-id=5-3055&mode=design) — match pixel-perfect (per the README).

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
- Match the [Figma reference](https://www.figma.com/file/nR9Voet7KCWpOYNJzB4O4Z/Frontend-Engineer-Interview-Exercise%3A-Employees-Page?type=design&node-id=5-3055&mode=design) **pixel-perfect** (per the README): layout, spacing, colors, typography, and row height.

## Open questions
- Exact design tokens (spacing, colors, typography, row height) need to be read from the Figma —
  via Dev Mode values or shared screenshots/exports — to hit pixel-perfect.
- Whether to show `inactive`/`trackingCategory` in the table or reserve for the detail panel.

## Out of scope
- Pagination, search, filters, detail panel (separate specs).
