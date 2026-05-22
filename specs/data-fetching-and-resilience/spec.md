# Data Fetching and Resilience

**Status:** Draft
**Last updated:** 2026-05-22
**Plan task:** 0.2

## Summary
A typed GraphQL data layer (TanStack Query + graphql-request) that survives the mock server's
chaos: random 5xx errors (~5%), 50–800 ms latency, and 60/min rate limiting with `Retry-After`.

## Motivation
Every screen reads from a deliberately unreliable API. A centralized, resilient fetch layer keeps
retry/backoff/timeout behavior consistent, prevents one flaky request from crashing a view, and
avoids self-throttling against the rate limit.

## Requirements
- `graphqlRequest` wrapper around graphql-request that reads `VITE_API_BASE`.
- Centralized retry policy: exponential backoff, max 3 retries, **honor `Retry-After` on 429**,
  do not retry 4xx other than 429.
- Typed interfaces: `Employee`, `Team`, `Account`, `EmployeeConnection`, `PageInfo`, `FilterOptions`.
- A forced 5xx retries, then surfaces a typed error (no crash / blank page).
- The 429 path waits the `Retry-After` interval — unit-tested on the backoff function.

## Open questions
- `staleTime` / `gcTime` defaults for query caching.
- Global error toast vs inline-only error surfacing.

## Out of scope
- Pagination, search, and filter UI (separate specs).
- Offline support / request queueing.
