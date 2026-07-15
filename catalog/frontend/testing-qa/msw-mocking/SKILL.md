---
name: msw-mocking
description: MSW conventions — per-feature handler organization, fail-on-unmocked-requests, per-test overrides, schema-validated fixtures shared between tests, dev, and Storybook.
---

# MSW (Mock Service Worker)

Network-level mocking: the real http-client and all its interceptor/
normalization code runs in tests — which is exactly why we don't mock the
client module (see http-client, testing-library-react).

## 1. Organization

```
src/mocks/
  handlers/orders.ts      # one file per feature/API area
  handlers/index.ts       # combines them: [...orderHandlers, ...authHandlers]
  fixtures/orders.ts      # data factories (below)
  server.ts               # setupServer(...handlers)  → tests
  browser.ts              # setupWorker(...handlers)  → dev/Storybook [adapt]
```

Default handlers return the HAPPY PATH. Error cases are per-test overrides,
not flags inside handlers.

## 2. Test wiring (in the shared setup file)

```ts
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

- `onUnhandledRequest: 'error'` is non-negotiable — an unmocked request
  failing loudly is how you find drift the day an endpoint is added
- `resetHandlers()` in afterEach — a leaked override is a classic
  order-dependent flake (fix-flaky-test cause #2)

## 3. Per-test overrides — error and edge cases

```ts
test('shows error state when refunds fail', async () => {
  server.use(http.get('*/refunds', () => HttpResponse.json(
    { error: { code: 'server_error', message: 'boom' } }, { status: 500 },
  )))
  // render, assert the error UI
})
```

Override responses must match the REAL API's error shape (the one
http-client normalizes) — inventing shapes tests code paths production
never sees.

## 4. Fixtures — factories, validated by the real schemas

- Factories with overrides, not inline literals:
  `makeRefund({ status: 'rejected' })`
- Fixtures are parsed with the app's zod schemas in a test
  (`refundSchema.parse(makeRefund())`) — when the API contract changes,
  the FIXTURE fails first, loudly, instead of every mock silently lying
- No `delay()` in shared handlers — latency simulation is per-test, or the
  whole suite pays for it

## 5. Reuse beyond tests

The same handlers power [adapt: dev mode without backend via browser.ts,
Storybook via msw-storybook-addon]. One mock definition per endpoint,
everywhere — two divergent mock sets is worse than none.
