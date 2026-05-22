# Implementation Plan: Faros AI — Employee Insights Dashboard

> Take-home: Lead Frontend (IC). Budget ~8 hrs. Rubric weights AI integration & safety,
> production thinking, and UX judgment **over** pixel-perfect styling — but the table
> Figma must still match. Plan reflects that priority order.

## Overview

A React SPA that renders a paginated, filterable employee table from a chaotic GraphQL
API, with an AI-insights panel that handles consent, PII contamination, low confidence,
and timeouts safely. Plus production scaffolding: telemetry, a runtime feature flag,
error boundaries, a RUNBOOK, and a DECISIONS doc.

## Locked architecture decisions

- **Stack:** React 18 + Vite + TypeScript, **shadcn/ui** (Radix + Tailwind) for accessible primitives.
- **Data layer:** **TanStack Query + graphql-request**. TanStack owns retry/backoff/timeout/caching;
  graphql-request is a thin typed fetch. This gives explicit control over the chaos behaviors the rubric tests.
- **Repo layout / submission repo:** the app lives at `faros-takehome/app/`, which **is** the
  submission git repo, wired to **`git@github.com:nikhil-venkat/ai-employee-insights.git`** (remote `origin`,
  branch `main`). The remote already exists with a `main` branch, so Task 0.1 **clones** it (not `git init`)
  to preserve its history. `DECISIONS.md`, `RUNBOOK.md`, `README.md` live at this repo root (`app/`).
  The mock server stays under `frontend-exercise-template-main/mock-server/` — **outside** `app/`, so it is
  naturally excluded from the submission. Every per-task commit (see commit cadence) is made in `app/` and
  pushed to `origin/main`; the surrounding `~/Desktop/projects` repo is never used for this work.
- **Tests:** Vitest + React Testing Library. Concentrate on **AI content validation** (PII redaction,
  confidence gating) and the filter/pagination reducer — not trivial component coverage.

## Server facts that drive the design (from reading the source)

| Behavior | Value | Plan implication |
|---|---|---|
| GraphQL errors | 5% random 5xx | Retry w/ exponential backoff (cap ~3), surface inline + telemetry |
| GraphQL latency | 50–800 ms | `keepPreviousData` on pagination/filter to avoid flicker |
| GraphQL rate limit | 60/min/IP, sends `Retry-After` | Backoff must respect `Retry-After`; debounce search to avoid self-throttle |
| AI latency | 200 ms–5 s | Skeleton/streaming-feel loading state, never a frozen button |
| AI timeout | 5% → 504 after 10–20 s | Client `AbortController` at ~8–10 s; graceful fallback + manual retry |
| AI PII contamination | **15%** of summaries | **Detect + redact before render**, warn user, emit `ai_pii_detected` telemetry |
| AI low confidence | **10%** return <0.30 | Show confidence; below threshold → caveat banner or suppress body |
| AI consent | token via POST /consent, **expires 1 hr**, `X-Consent-Token` header; 401 missing / 403 invalid/expired | Explicit consent UI (human-in-loop); handle expiry with re-consent |
| AI rate limit | 10/min/IP (separate from GraphQL) | Disable rapid re-requests; respect `Retry-After` |

## Data-quality landmines in `employees.json` (32 rows — verified)

- `emp_18` (arthur): **name is `null`** → fall back to uid; never render "null".
- `emp_25` (voldemort): **no email, no photo, no teams, no accounts, trackingCategory null** → every cell must degrade gracefully.
- `emp_21` (cedric), `emp_25`: **no accounts** → "Accounts Connected" empty state.
- `emp_30` (padma), `emp_25`: **no teams** → "—" / "Unassigned".
- `emp_13` (george), `emp_25`: **no email**.
- `emp_24` (bellatrix), `emp_25`: **trackingCategory null**.
- `photoUrl` is dicebear (may 404 / be slow) → avatar needs initials fallback on image error.

Account types: `vcs`(GitHub), `tms`(Jira), `ims`(PagerDuty), `cal`(Google Calendar) — icons in `assets/icons/`.
Teams: backend, collaboration, data-platform, frontend, infrastructure, sales, security.

---

## Specs by phase (implementation order)

Each PLAN task is tracked as a spec under `app/specs/<name>/`. Build order follows the dependency
graph — foundation → dashboard slices → AI → hardening — and each spec ships as its own branch → PR.

**Phase 0 — Foundation**
1. `app-scaffold` (0.1) — prereq for everything
2. `data-fetching-and-resilience` (0.2) — every screen reads through it
3. `error-boundaries` (0.3) · `telemetry` (0.3) · `feature-flags` (0.3) — parallelizable trio; primitives built here, applied throughout, finalized in Phase 3

**Phase 1 — Dashboard** (on the data layer)
4. `employee-table` (1.1) — host for the rest
5. `cursor-pagination` (1.2) · `employee-search` (1.3) · `employee-filters` (1.4) — siblings off the table
6. `employee-detail-panel` (1.5) — host for the AI feature

**Phase 2 — AI Insights** (on detail panel + feature-flags)
7. `ai-consent-flow` (2.1)
8. `ai-insights-fetch` (2.2)
9. `ai-content-safety` (2.3) — ⭐ highest-signal: PII redaction + confidence gating

**Phase 3 — Hardening + docs**
10. `telemetry` instrumentation pass (3.1) + `error-boundaries` placement review (3.2)
11. `RUNBOOK.md` (3.3) · `DECISIONS.md` + `README.md` (3.4) — docs, not specs

**Critical path:** scaffold → data layer → table → detail panel → consent → insights → content-safety.
**Woven throughout:** telemetry, error boundaries, feature flags (built early, applied as features land, finalized in Phase 3).

---

## Task List

### Phase 0 — Foundation

#### Task 0.1: Wire submission repo + scaffold app + tooling
**Description:** Stand up the submission repo and the app inside it.
1. **Clone the existing remote** into the app dir — it already has a `main` with history, so don't `git init`:
   `git clone git@github.com:nikhil-venkat/ai-employee-insights.git app` (run from `faros-takehome/`).
   Confirm `origin` → `git@github.com:nikhil-venkat/ai-employee-insights.git` and branch `main`.
2. **Scaffold Vite React-TS into `app/`.** Because `app/` is non-empty (cloned `.git` + any README/.gitignore),
   scaffold to a temp dir (`npm create vite@latest .tmp-scaffold -- --template react-ts`) and copy files into `app/`,
   preserving `.git` and merging any existing README. Then Tailwind + shadcn/ui init, Vitest + RTL, ESLint/Prettier.
3. **Env + ignore:** add `.env` with `VITE_API_BASE=http://localhost:4000` and a committed `.env.example`;
   verify Vite's `.gitignore` covers `node_modules/`, `dist/`, `.env` (add if missing — never commit `.env`).
4. **Assets:** copy the four account icons + faros logo from `../frontend-exercise-template-main/assets/icons/` into `app/src/assets/`.
5. **Initial commit + push:** commit the scaffold and `git push -u origin main` to set upstream and prove the pipe works.
**Acceptance criteria:**
- [ ] `app/` is a git repo whose `origin` is the ai-employee-insights remote, tracking `origin/main`; remote history preserved (no force-push).
- [ ] `npm run dev` serves a blank shell; `npm run build` and `npm test` both succeed.
- [ ] shadcn `button` + one primitive render; Tailwind classes apply.
- [ ] `.env` is gitignored (not in `git status`); `.env.example` is tracked.
- [ ] Scaffold commit is visible on `origin/main` after push.
**Verification:** `git remote -v` + `git status -sb`; `npm run build`, `npm test`; dev server at :5173; `git log origin/main` shows the scaffold commit.
**Dependencies:** None. **Files:** project scaffold (~scaffold) in `app/`. **Scope:** S.

#### Task 0.2: Data layer + chaos-aware fetch
**Description:** `QueryClientProvider`; a `graphqlRequest` wrapper around graphql-request reading `VITE_API_BASE`. Centralize retry policy: exponential backoff, max 3, **honor `Retry-After` on 429**, don't retry 4xx (except 429). Typed TS interfaces for `Employee`, `Team`, `Account`, `Connection`, `FilterOptions`.
**Acceptance criteria:**
- [ ] A throwaway `health`/employees query renders count in the shell.
- [ ] Forced 5xx (set `GRAPHQL_ERROR_RATE=1` in a scratch run) retries then surfaces a typed error, not a crash.
- [ ] 429 path waits the `Retry-After` interval (unit-tested on the backoff fn).
**Verification:** unit test for backoff/`Retry-After`; manual high-error-rate run.
**Dependencies:** 0.1. **Files:** `src/lib/graphql.ts`, `src/lib/retry.ts`, `src/types.ts`, `src/lib/queryClient.ts`. **Scope:** M.

#### Task 0.3: Cross-cutting infra — error boundary, telemetry, feature flag
**Description:** (a) Root + section-level `ErrorBoundary` (table vs AI panel isolated). (b) `telemetry.ts` client → `POST /api/telemetry`, batched, `navigator.sendBeacon`/`fetch keepalive` on unload, never throws. (c) `featureFlags.ts` provider: precedence **URL param > localStorage > default**, exposing `aiInsights` flag — flippable at runtime without redeploy.
**Acceptance criteria:**
- [ ] Throwing a test error in the AI section shows a fallback while the table stays alive.
- [ ] A telemetry event appears at `GET /api/telemetry` (evaluator endpoint).
- [ ] `?ff_aiInsights=off` hides the AI feature; reload via localStorage persists.
**Verification:** manual toggle + telemetry read; unit test flag precedence.
**Dependencies:** 0.1. **Files:** `src/components/ErrorBoundary.tsx`, `src/lib/telemetry.ts`, `src/lib/featureFlags.ts`. **Scope:** M.

### ✅ Checkpoint A (after 0.1–0.3)
- [ ] Build/test/lint clean. Data layer survives forced errors. Telemetry + flag + boundary proven. Commit.

---

### Phase 1 — Employee Dashboard (Part 1) — vertical slices

> **Blocked on Figma screenshots for final styling.** Build structure/behavior first;
> apply pixel spacing/colors once screenshots arrive.

#### Task 1.1: Employee table — fetch + columns + degraded data
**Description:** `employees` query → table with **Name (avatar + initials fallback)**, **Tracking Status** (badge), **Teams** (chips, "Unassigned" when empty), **Accounts Connected** (type icons, empty state). Loading skeleton, error (with retry), empty states. Name falls back to uid; never renders "null".
**Acceptance criteria:**
- [ ] All 32 rows render; `arthur`/`voldemort` degrade with no "null"/blank crash.
- [ ] Avatar falls back to initials when `photoUrl` fails.
- [ ] Loading → skeleton; fetch error → inline retry, not blank page.
**Verification:** load app; throttle/break image; RTL test for null-name + no-accounts rows.
**Dependencies:** 0.2. **Files:** `src/features/employees/EmployeeTable.tsx`, `Avatar.tsx`, `AccountIcons.tsx`, `useEmployees.ts`. **Scope:** M.

#### Task 1.2: Cursor pagination
**Description:** Next/Prev (or "Load more") on Relay cursors via `pageInfo`/`endCursor`, `keepPreviousData` to avoid flicker, page-size control, `totalCount` display. Disable controls at boundaries.
**Acceptance criteria:**
- [ ] Paging forward/back keeps stable rows, no flash; respects `hasNextPage`.
- [ ] Works combined with active search/filters (cursor resets on criteria change).
**Verification:** manual page through 32 rows at pageSize 10; RTL for cursor reset on filter change.
**Dependencies:** 1.1. **Files:** `EmployeeTable.tsx`, `useEmployees.ts`, `Pagination.tsx`. **Scope:** S.

#### Task 1.3: Search (debounced)
**Description:** Name search box, ~300 ms debounce (protects the 60/min limit), resets cursor, shows "no results", clearable.
**Acceptance criteria:**
- [ ] Typing "pot" filters to Harry Potter; clearing restores; debounce verified (no per-keystroke request).
**Verification:** manual + Network panel; unit test on debounce hook.
**Dependencies:** 1.1. **Files:** `SearchInput.tsx`, `useEmployees.ts`. **Scope:** S.

#### Task 1.4: Filters (team, tracking status, account type)
**Description:** Populate dropdowns from `filterOptions` query. Multi-select team / accountType / trackingStatus, combine with search, reset cursor, "clear all", reflect active filters.
**Acceptance criteria:**
- [ ] Filtering by team `frontend` + accountType `ims` narrows correctly and composes with search.
- [ ] Filter options come from the API, not hardcoded.
**Verification:** manual cross-filter; RTL on filter-state composition.
**Dependencies:** 1.1, (1.3 for compose). **Files:** `Filters.tsx`, `useFilterOptions.ts`, `useEmployees.ts`. **Scope:** M.

#### Task 1.5: Employee detail panel ("View")
**Description:** "View" opens a Sheet/Drawer (Radix, focus-trapped, Esc-closable) showing name, email, teams, accounts, tracking, activity. Degrades for missing fields. This is the **host for the AI panel**.
**Acceptance criteria:**
- [ ] View opens panel for any row incl. degraded ones; keyboard accessible; closes cleanly.
**Verification:** manual + a11y keyboard pass.
**Dependencies:** 1.1. **Files:** `EmployeeDetailPanel.tsx`, `useEmployee.ts`. **Scope:** M.

### ✅ Checkpoint B (after 1.1–1.5)
- [ ] Full Part-1 flow works end-to-end against the chaotic server. Apply Figma screenshots → pixel pass. Browser-verify. Commit.

---

### Phase 2 — AI Employee Insights (Part 2)

#### Task 2.1: Consent flow (human-in-the-loop)
**Description:** Before any insight call, explicit consent UI explaining what's shared/why. On accept → `POST /api/ai/consent` `{userId, scope:"insights"}`, store token + `expiresAt` in memory/session. Handle expiry → re-consent. Gated by `aiInsights` feature flag (Task 0.3).
**Acceptance criteria:**
- [ ] No insight request fires without a valid token. Expired token triggers re-consent, not a raw 403.
- [ ] Flag off → AI section absent entirely.
**Verification:** manual consent + forced-expiry; telemetry `ai_consent_granted`.
**Dependencies:** 0.3, 1.5. **Files:** `features/ai/consent.ts`, `ConsentGate.tsx`, `useConsent.ts`. **Scope:** M.

#### Task 2.2: Insights fetch — latency, timeout, retry
**Description:** `GET /api/ai/insights/:id` with `X-Consent-Token`. Loading state for up to 5 s (skeleton + "generating", not a frozen spinner). **Client `AbortController` timeout ~8–10 s** → friendly timeout fallback + manual "Try again" (respect 10/min limit, surface `Retry-After`). Handle 401/403/404/429/504 distinctly.
**Acceptance criteria:**
- [ ] Normal call renders summary + metadata (model, processingTime, generatedAt).
- [ ] Forced timeout (`AI_TIMEOUT_RATE=1`) aborts client-side with fallback, no infinite spinner.
- [ ] Each status code maps to a distinct, non-crashing message.
**Verification:** scratch runs flipping `AI_TIMEOUT_RATE`/`AI_RATE_LIMIT`; telemetry on each outcome.
**Dependencies:** 2.1. **Files:** `features/ai/insights.ts`, `useInsights.ts`, `InsightPanel.tsx`. **Scope:** M.

#### Task 2.3: Content safety — PII redaction + confidence gating  ⭐ highest-signal task
**Description:** Validate AI text **before render**. (a) **PII detection/redaction**: regex for phone, SSN, email, address, DOB → redact (mask) + show a "sensitive content removed" warning; emit `ai_pii_detected`. (b) **Confidence**: always show score; `< 0.3` → caveat banner ("low confidence, verify independently") and de-emphasize/optionally collapse the body. (c) Feedback control (👍/👎) → telemetry.
**Acceptance criteria:**
- [ ] Injected PII (all 5 snippet types) is redacted and never reaches the DOM verbatim — **unit tested per type**.
- [ ] confidence 0.2 shows the low-confidence treatment; 0.9 shows normal.
- [ ] Thumbs feedback emits telemetry.
**Verification:** **unit tests are the deliverable here** — feed known PII/confidence fixtures through the validator; manual feedback click.
**Dependencies:** 2.2. **Files:** `features/ai/contentSafety.ts`, `contentSafety.test.ts`, `InsightPanel.tsx`, `ConfidenceBadge.tsx`, `FeedbackButtons.tsx`. **Scope:** M.

### ✅ Checkpoint C (after 2.1–2.3)
- [ ] Consent → insight → safe render works through all failure modes. PII tests green. Browser-verify forced PII + timeout. Commit.

---

### Phase 3 — Production Readiness & Docs (Part 3 + 4)

#### Task 3.1: Telemetry instrumentation pass
**Description:** Wire the event taxonomy across flows: `page_view`, `employee_search`, `employee_filter`, `employee_paginate`, `employee_view`, `ai_consent_granted`, `ai_insight_requested/succeeded/failed/timeout`, `ai_pii_detected`, `ai_low_confidence`, `ai_feedback`. Consistent envelope (event, ts, sessionId, props). **Never log PII or raw AI text** in telemetry.
**Acceptance criteria:**
- [ ] Each user action produces exactly one well-formed event at `GET /api/telemetry`.
- [ ] No PII/summary text in any payload (asserted in review).
**Verification:** exercise flows, read evaluator telemetry endpoint.
**Dependencies:** 0.3 + features. **Files:** `src/lib/telemetry.ts`, call sites. **Scope:** S–M.

#### Task 3.2: Error-boundary + degradation hardening
**Description:** Confirm boundaries isolate table vs AI panel vs filters; add retry affordances; final empty/error/loading polish; ensure a thrown AI error never blanks the table.
**Acceptance criteria:**
- [ ] Injected throw in AI panel leaves table interactive; injected throw in a row cell is contained.
**Verification:** manual fault injection.
**Dependencies:** all features. **Files:** boundary placements. **Scope:** S.

#### Task 3.3: RUNBOOK.md
**Description:** One-page triage guide for the AI insights feature in prod: symptoms → checks in order (consent token issuance, AI endpoint health, latency/timeout rate, PII-redaction alarm, rate-limit 429s, feature-flag kill switch), key metrics/dashboards, and the **flag-off kill switch** as first mitigation.
**Acceptance criteria:**
- [ ] Fits ~1 page; ordered checks; names the metrics from our telemetry; states the rollback/kill switch.
**Verification:** self-review against rubric dimension 3.
**Dependencies:** 3.1. **Files:** `RUNBOOK.md`. **Scope:** S.

#### Task 3.4: DECISIONS.md + submission README
**Description:** Cover all 6 prompts: architecture/tradeoffs; **AI dev workflow & environment** (Claude Code setup, this PLAN.md, skills, where AI output needed correction, what I'd change for a long-lived project); data/API challenges (the landmine table above); privacy/security (PII redaction, consent, no-PII telemetry, token handling); what's-missing-for-prod; testing strategy incl. **testing AI-generated content**. Plus a setup README noting the mock server must run at :4000 and is not included.
**Acceptance criteria:**
- [ ] All six DECISIONS sections present and specific; README runs clean on a fresh clone (with server caveat).
**Verification:** fresh-clone mental walkthrough; cross-check against README rubric.
**Dependencies:** everything. **Files:** `DECISIONS.md`, `README.md`. **Scope:** M.

### ✅ Checkpoint D — Submission ready
- [ ] All four parts complete. `build`+`test`+`lint` green. Telemetry verified. Flag kill-switch works. Docs complete. Mock server excluded. Final browser pass. Push to private repo.

---

## Risks & mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Figma screenshots delayed | Med | Build behavior first (Phase 0–1 don't need them); apply styling at Checkpoint B |
| Chaos flakiness during dev/demo | Med | Env knobs to temporarily zero error/timeout rates in scratch runs; never commit lowered rates |
| AI rate limit (10/min) hit while testing | Med | Throttle re-requests in UI; cache last insight per employee in-session |
| PII regex misses a variant | High (safety) | Test all 5 injected snippet types; default-deny tone in copy ("may contain errors") |
| Over-scoping styling vs AI/prod | Med | Time-box Part 1 polish; rubric values AI+prod over pixels |

## Open questions

- **userId for consent**: no auth — use a synthetic/session `userId` (e.g. `demo-user`). Confirm acceptable; document in DECISIONS.
- **Feature-flag source**: URL/localStorage is enough to demo "no redeploy." A polled config endpoint is nicer but likely over-budget — confirm the simple version is fine.
- **Insight caching**: cache per employee for the session to dodge rate limits, with a manual "regenerate"? (Recommended.)

## Suggested commit cadence
All work happens in `app/` against **`origin = git@github.com:nikhil-venkat/ai-employee-insights.git`**.
One commit per task on `main`; **push at each checkpoint** (the natural review points) so `origin/main`
always reflects a working state. Task 0.1 establishes the repo and upstream; every later task commits and
pushes there. The surrounding `~/Desktop/projects` repo is never touched.
