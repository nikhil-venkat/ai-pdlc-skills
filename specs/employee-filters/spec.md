# Employee Filters

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 1.4

## Summary
Multi-select filters for team, tracking status, and account type, populated from the
`filterOptions` query and composable with search and pagination.

## Motivation
Part 1 requires filtering by team, tracking status, and account type. Options must come from the
API (not hardcoded) so they stay correct as data changes.

## Requirements
- Dropdowns populated from `filterOptions` (teams, trackingStatuses, accountTypes).
- Multi-select for team / accountType / trackingStatus; combine with active search.
- "Clear all"; active filters are visible; cursor resets on change.
- Example: team `frontend` + accountType `ims` narrows correctly and composes with search.

## Open questions
- Whether to also expose `trackingCategories` (Active/Inactive) as a filter (schema supports it).

## Out of scope
- Saved filter presets; URL-encoding the full filter state (nice-to-have).
