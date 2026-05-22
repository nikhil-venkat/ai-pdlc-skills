# Employee Table

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.1
**Phase:** 1 — Dashboard
**Design reference:** [Figma — Employees Page](https://www.figma.com/file/nR9Voet7KCWpOYNJzB4O4Z/Frontend-Engineer-Interview-Exercise%3A-Employees-Page?type=design&node-id=5-3055&mode=design) — match pixel-perfect (per the README). Frame exports + manifest: [`design/`](./design/README.md) (9 frames).

## Summary
The core employee table: fetch from the GraphQL `employees` query and render Name (avatar),
Tracking Status, Teams, and Accounts Connected — degrading gracefully on dirty data.

## Motivation
This is the primary view of Part 1 and the host for every other dashboard feature (search,
filters, pagination, detail panel). It must look professional and never crash on the dataset's
known data-quality landmines.

## Requirements

### Layout (from Figma — see [`design/`](./design/README.md))
- **Page shell:** top nav (workspace switcher, Modules/Scorecard, Personal/Acme icons), breadcrumb
  *Admin Settings › Organization Setup › Employees Page*, "Employees" title + subtitle, and a **+ New**
  button (top-right, dark navy). Build a faithful header; the table is the priority.
- **Search:** full-width input "Search employees by name …" with a leading magnifier icon.
- **Filter bar:** **+ Add Filter** button; applied filters render as removable chips (e.g. "Team: Data Platform ✕").

### Table columns (left → right)
1. **Select** checkbox (header + per row).
2. **Name** — circular avatar (initials fallback on image error) + teal name link + gray email beneath. Falls back to uid when name is null; never renders "null".
3. **Tracking Status** — person icon + primary label ("Included"/"Ignored") + secondary ("Active"/"Inactive"). Icon color: **green** = Included·Active, **pink/red** = Included·Inactive, **gray** = Ignored.
4. **Teams** — colored chips, per-team colors (backend = teal, data-platform = orange, frontend = purple, Sales = green); "Unassigned" / "—" when empty.
5. **Accounts Connected** — source icons: Jira (blue diamond), GitHub, PagerDuty ("pd"), Google Calendar; empty state when none.
6. **View** — outlined button per row (opens the detail panel).
- Sortable headers show a ↓ arrow (Name, Tracking Status, Teams, Accounts Connected).

### Behavior
- All 32 rows render. Degraded rows (`arthur` null name, `voldemort` missing email/photo/teams/
  accounts) render with **no "null" or blank crash**.
- **Pagination** footer, bottom-right: "1–5 ▾ of N" with page-size dropdown and ‹ › chevrons (detailed in the `cursor-pagination` spec).
- Loading → skeleton; fetch error → inline retry (not a blank page); empty result → empty state.
  *(These states are not in the Figma — they're our production-readiness additions.)*

## Open questions
- Exact design tokens (px spacing, hex colors, font sizes, row height) still need Figma Dev Mode
  values; the frame PNGs give layout/proportions but not precise tokens.
- Whether to show `inactive`/`trackingCategory` in the table or reserve for the detail panel.
- Figma's detail panel is an **editable** Profile Info form (Save/Cancel), but our API is read-only —
  resolved in the `employee-detail-panel` spec (panel will be read-only + AI host).

## Out of scope
- Pagination, search, filters, detail panel — separate specs (the Figma frames for those live in
  [`design/`](./design/README.md) and are cross-referenced from those specs).
