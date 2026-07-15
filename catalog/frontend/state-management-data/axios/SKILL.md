---
name: axios
description: Axios conventions — one instance, interceptor order, method-aware retry (idempotent-only, never blind POST retry), Retry-After respect, and the double-retry trap.
---

# Axios

The concrete implementation of the http-client skill for axios repos —
install both; http-client owns the architecture (ApiError shape,
single-flight refresh), this owns the axios mechanics.

## 1. One instance

```ts
export const api = axios.create({
  baseURL: env.API_BASE_URL,
  timeout: 10_000,                      // [adapt] — no request hangs forever
  headers: { 'Content-Type': 'application/json' },
})
```

Feature code imports `api` (via the http-client wrapper) — a bare
`axios.get(...)` anywhere is the review catch.

## 2. Interceptors — minimal, ordered

- Request: auth header injection, request-id header (ties to backend
  correlation — logging conventions)
- Response error: normalize EVERYTHING to `ApiError` (http-client §1) and
  rethrow; the 401→single-flight-refresh→retry-once flow lives here too
- That's it. Business logic in interceptors is invisible action at a
  distance — it fails review

## 3. Retry — method-aware or it's a bug factory

[adapt: axios-retry or the hand-rolled equivalent]

```ts
axiosRetry(api, {
  retries: 2,
  retryDelay: (n) => axiosRetry.exponentialDelay(n),   // backoff + jitter
  retryCondition: (err) =>
    axiosRetry.isNetworkError(err) ||
    (isIdempotent(err.config) && is5xxOr429(err)),
})
```

- **GET/HEAD (and app-idempotent PUT/DELETE [decide — adapt]): retryable**
  on network errors, 5xx, 429
- **POST: NEVER auto-retried** — "did the refund submit twice?" is this
  exact bug. A POST that must survive retries sends an idempotency key
  header the backend honors [adapt: `Idempotency-Key`] — then and only
  then it may retry
- Respect `Retry-After` on 429; cap total attempts (2–3) — retry storms
  amplify outages
- 4xx (except 429): never retried — the request is wrong, not unlucky

## 4. The double-retry trap (decide the owner)

TanStack Query ALSO retries. Two layers retrying = up to attempts²
requests during an outage. House rule [adapt]: **queries retry at the
Query layer (retry: 1–2), axios-retry handles only network-level errors
for mutations' idempotent cases** — one owner per failure class, written
in http-client's config comment.

## 5. Cancellation & tests

- Forward `AbortSignal` (`api.get(url, { signal })`) — TanStack Query
  passes it; dropping it leaks in-flight requests on unmount/navigation
- Tests through MSW (msw-mocking): retry test = handler fails once then
  succeeds, assert exactly 2 requests and one result; POST test asserts
  exactly ONE request on failure
