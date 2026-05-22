# Employee Table — Figma frame reference

Source: [Employees Page Figma](https://www.figma.com/file/nR9Voet7KCWpOYNJzB4O4Z/Frontend-Engineer-Interview-Exercise%3A-Employees-Page?type=design&node-id=5-3055&mode=design).
Exported as PNG. Filenames are the raw Figma export names; descriptions below.

These frames cover the whole **Employees Page** (Part 1) — the table plus its filter flow and the
detail panel — so they also inform the `employee-filters` and `employee-detail-panel` specs.

| File | Frame | What it shows |
|------|-------|---------------|
| `Frame.png`   | Default table | Employees table, no filters: top nav + breadcrumb (Admin Settings › Organization Setup › Employees Page), "Employees" title + subtitle, **+ New** button, search bar, **+ Add Filter**, 5 rows, pagination "1–5 ▾ of 32". |
| `Frame-1.png` | Table + applied chip | Same table with an applied **"Team: All ✕"** filter chip beside Add Filter. |
| `Frame-2.png` | Filtered: Data Platform | Table filtered to Team = Data Platform; chip "Team: Data Platform ✕"; pagination "1–5 of 11". |
| `Frame-3.png` | Detail panel (View) | Right drawer for *Avery Erickson*: "Included · Active" status pill, external-link + close (✕), **Profile Info** form (UID, Title, Name, Email, Role, Location, Level, Employment Type) with **Save / Cancel**. Table shrinks with a horizontal scrollbar. ⚠️ Editable form — see notes. |
| `Frame-4.png` | Team filter — all selected | Team multi-select dropdown: search box, **Deselect all**, Backend/Data Platform/Frontend/Sales all checked, Apply/Cancel. |
| `Frame-5.png` | Team filter — none selected | Same dropdown with nothing checked, **Select all**. |
| `Frame-6.png` | Team filter — one selected | Same dropdown, only **Data Platform** checked. |
| `Frame-7.png` | Add Filter menu | "FILTER BY" type picker: **Accounts Connected / Team / Tracking Status**, Apply/Cancel. |
| `Frame-8.png` | Add Filter — Team checked | Same picker with **Team** checked (step before the Team dropdown opens). |

## Observed design details (for pixel-perfect build)

- **Columns:** select checkbox · **Name** (circular avatar + teal name link + gray email) · **Tracking
  Status** (person icon + "Included/Ignored" + secondary "Active/Inactive") · **Teams** (colored chips) ·
  **Accounts Connected** (source icons) · **View** (outlined button). Sortable headers show a ↓ arrow.
- **Tracking status icon colors:** green person = Included·Active; pink/red person = Included·Inactive;
  gray person = Ignored.
- **Team chip colors are per-team:** backend = teal, data-platform = orange, frontend = purple, Sales = green.
- **Account icons:** Jira (blue diamond), GitHub (octocat), PagerDuty ("pd" green), Google Calendar (color square).
- **Pagination:** bottom-right, "1–5 ▾ of 32" with ‹ › chevrons and a page-size dropdown.
- **Filter flow:** Add Filter → pick a filter type → (for Team) multi-select dropdown with search +
  Select/Deselect all → applied filters render as removable chips ("Team: … ✕").

## Notes / divergences from our implementation

- **Detail panel is an *editable* Profile Info form** (Save/Cancel, "Add Title/Role/Location/Level").
  Our GraphQL API is **read-only** (no mutations), so editing is **out of scope** unless we add it; the
  panel we build is read-only and also hosts the AI insights feature. Tracked in `employee-detail-panel`.
- **No loading / empty / error frames** were provided. Those states are our own production-readiness
  additions (skeleton, inline retry, empty state), not from Figma.
- **Top-nav chrome** (Workspace switcher, Modules/Scorecard, Personal/Acme icons) is app shell context;
  build a faithful-enough header but the table + filters + panel are the priority.
