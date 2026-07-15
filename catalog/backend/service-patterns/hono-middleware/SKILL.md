---
name: hono-middleware
description: Writing custom Hono middleware — createMiddleware typing, the await-next contract, before/after phases, short-circuiting, error behavior, isolated testing.
---

# Hono middleware

hono-patterns §2 defines the ORDER; this skill is how to WRITE one
correctly. Middleware is for cross-cutting concerns (auth, request-id,
timing, tenant resolution) — business logic in middleware is invisible
action at a distance and fails review (same rule as axios interceptors).

## 1. The shape and the contract

```ts
import { createMiddleware } from 'hono/factory'

export const timing = createMiddleware<{ Variables: { user: AuthUser } }>(
  async (c, next) => {
    const start = performance.now()          // BEFORE phase
    await next()                             // exactly once — the contract
    c.header('Server-Timing', `app;dur=${performance.now() - start}`)  // AFTER
  },
)
```

- `createMiddleware` with the app's typed `Variables` — `c.set`/`c.get`
  stay typed (hono-patterns §5)
- **`await next()` exactly once.** Forgetting `await` = the after-phase
  runs before downstream finishes (timing lies, headers race); calling it
  twice = handlers execute twice
- After-phase code must not throw over the real response — wrap
  nice-to-have after-work in its own try/catch

## 2. Short-circuiting (auth's pattern)

```ts
export const requireRole = (role: Role) => createMiddleware(async (c, next) => {
  const user = c.get('user')
  if (!user) throw new HTTPException(401, { message: 'unauthenticated' })
  if (!can(user, role)) throw new HTTPException(403, { message: 'forbidden' })
  await next()
})
```

- Reject by THROWING `HTTPException` — one error path through `onError`
  (hono-patterns §4), not `return c.json(...)` ad-hoc 401 shapes
- Factory form (`(config) => middleware`) for parameterized middleware —
  not env-reading singletons

## 3. Context discipline

- `c.set` in middleware, `c.get` in handlers — handlers never set
- Set VALIDATED, typed values (the parsed user, the resolved tenant), not
  raw headers for handlers to re-parse
- One middleware = one variable it owns; two middleware writing the same
  key is a design bug

## 4. Errors in middleware

Let unexpected errors propagate to `onError` — a middleware try/catch that
logs-and-continues turns auth failures into open doors. Catch ONLY what
the middleware genuinely handles (e.g. optional-enrichment lookup fails →
proceed without it, log warn).

## 5. Testing — isolated, with a mini app

```ts
const app = new Hono()
app.use(requireRole('admin'))
app.get('/t', (c) => c.json({ ok: true }))
// assert: no token → 401 body shape; wrong role → 403; admin → 200
```

Each middleware gets its own spec: happy path, each rejection, and (for
after-phase middleware) that downstream errors still produce the standard
error shape.
