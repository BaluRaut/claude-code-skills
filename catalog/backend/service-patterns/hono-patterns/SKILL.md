---
name: hono-patterns
description: Hono on Lambda conventions — app/router structure, middleware order, zod validation everywhere, the HTTPException error taxonomy, context typing.
---

# Hono patterns

Adapter: `hono/aws-lambda`, one catch-all `api` function unless a route has
different scaling/permission needs (see serverless-v3-config).

## 1. Structure

- `src/app.ts` — the app: global middleware + `app.route()` registrations +
  ONE `onError` + ONE `notFound`. Nothing else
- `src/routes/<resource>.ts` — one router per resource; routes stay thin:
  validate → call service function → shape response
- Business logic in `src/services/` — plain functions, no Hono types, so
  they're testable and reusable from queue/cron handlers

## 2. Middleware — order is behavior

```ts
app.use(requestId())          // 1. correlation id first (logging needs it)
app.use(loggerMiddleware)     // 2. request log w/ id (logging-observability)
app.use(cors({ origin: env.ALLOWED_ORIGINS }))  // 3. never '*' with credentials
app.use('/admin/*', requireRole('admin'))       // 4. auth scoped by path
```

Route-specific middleware at the router, not buried in handlers.

## 3. Validation — every input, no exceptions

`zValidator('json' | 'query' | 'param', schema)` on every route that reads
input (schemas per zod conventions — shared with frontend if you have a
contracts package [adapt]). `c.req.valid()` is the ONLY way handlers read
input; `await c.req.json()` in a handler is the review catch.

## 4. Errors — one taxonomy

- Throw `HTTPException(status, { message })` for expected failures
  (404/403/409); messages are safe-for-client — internals never leak
- Unexpected errors: let them reach `onError` — it logs with the request id,
  reports to monitoring, returns `{ error: { code: 'internal', message } }`
  with a 500. Handlers do NOT try/catch-and-500 themselves
- Error body shape `{ error: { code, message, details? } }` matches what
  the frontend http-client normalizes — changing it breaks every app

## 5. Context typing

Type context variables once: `new Hono<{ Variables: { user: AuthUser } }>()`
— `c.get('user')` is typed everywhere; `c.set` from middleware only.

## 6. Verify

Route tests via `app.request()` (node-testing): happy path, a validation
400, an auth 401/403, and one thrown-through 500 asserting the error shape.
