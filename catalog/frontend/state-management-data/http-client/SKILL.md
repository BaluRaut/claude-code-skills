---
name: http-client
description: The single HTTP client under all data hooks — auth header injection, single-flight 401 refresh, normalized ApiError shape, timeouts and cancellation, zod at the boundary.
---

# HTTP client

One client module [src/api/client.ts — adapt: fetch wrapper or axios
instance]. Every request in the app goes through it; components and even
most hooks never touch `fetch`/`axios` directly (see new-data-hook). This
is the layer that makes "every hook handles errors the same way" true.

## 1. Responsibilities (all of them, only here)

- `baseURL` from validated env (env.ts), never hardcoded
- **Auth**: attach the token/credentials [adapt: header from auth module /
  cookie mode]; on 401 → refresh flow (below); auth logic exists NOWHERE else
- **Error normalization**: every failure becomes one typed shape —

```ts
export class ApiError extends Error {
  status: number          // 0 for network failure
  code: string            // machine-readable: 'validation_failed', 'not_found'
  details?: unknown       // field errors etc.
}
```

  Query hooks, forms (server-error mapping in new-form), and monitoring all
  rely on this shape — changing it is a breaking change
- **Timeouts**: default per request (`AbortSignal.timeout` / axios timeout)
  [value — adapt]; no request may hang forever
- **Cancellation**: accept and forward an `AbortSignal` — TanStack Query
  passes one per query; wire it through, don't drop it
- Response parsing: zod at this boundary (zod-schemas) [adapt: per-call
  schemas passed in, or parsing in the data layer above]

## 2. The 401 refresh — single-flight, or bugs

Ten parallel queries hitting an expired token must trigger ONE refresh:

```ts
let refreshing: Promise<void> | null = null
async function ensureFresh() {
  refreshing ??= doRefresh().finally(() => { refreshing = null })
  return refreshing
}
```

Retry the original request once after refresh; refresh failed → central
logout (auth-patterns), never a retry loop.

## 3. What the client does NOT do

- **Retries** — that's TanStack Query's policy; retrying in both layers
  multiplies requests
- **Caching** — same reason
- Swallowing errors: `catch (e) { return null }` in the client makes every
  hook's error state unreachable — normalize and RETHROW, always
- Business logic, response massaging beyond parse — that belongs in the
  data layer/hooks

## 4. Testing

Test THROUGH the client with MSW (msw-mocking) — never mock the client
module itself in feature tests; the interceptor/refresh/normalization code
is exactly what needs exercising. Direct client tests cover: 401→refresh→
retry once, refresh failure→logout, timeout→ApiError(status 0 or 408),
abort propagation.
