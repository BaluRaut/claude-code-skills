---
name: new-endpoint
description: Add a new HTTP endpoint to this Hono service — router, zod validation, registration, tests, and serverless.yml wiring when a new Lambda is needed.
---

# Add a new endpoint

Follow these steps in order. Do not skip the test step.

## 1. Locate or create the router

An endpoint for an existing resource goes into its file in `src/routes/`.
A new resource gets a new file following this shape:

```ts
// src/routes/refunds.ts
import { Hono } from 'hono'
import { zValidator } from '@hono/zod-validator'
import { z } from 'zod'
import { HTTPException } from 'hono/http-exception'

const createRefundSchema = z.object({
  orderId: z.string().uuid(),
  amountCents: z.number().int().positive(),
  reason: z.enum(['damaged', 'wrong_item', 'other']),
})

export const refunds = new Hono()
  .post('/', zValidator('json', createRefundSchema), async (c) => {
    const body = c.req.valid('json')
    // business logic here — throw HTTPException(404/409/...) for error cases
    return c.json({ id: refund.id }, 201)
  })
  .get('/:id', async (c) => {
    const refund = await getRefund(c.req.param('id'))
    if (!refund) throw new HTTPException(404, { message: 'refund not found' })
    return c.json(refund)
  })
```

Rules:
- Input validation is **always** zod via `zValidator` — `json` for bodies,
  `query` for query params, `param` for path params.
- Errors are `HTTPException` only; the global `onError` handles formatting.
- Log with `src/lib/logger.ts`, never `console.log`.

## 2. Register the router (new resource only)

In `src/app.ts`:

```ts
import { refunds } from './routes/refunds'
app.route('/refunds', refunds)
```

The catch-all `api` Lambda (`httpApi: '*'` in serverless.yml) picks it up —
**no serverless.yml change is needed for a plain HTTP endpoint.**

## 3. serverless.yml — ONLY for non-HTTP triggers

Only if the work item needs a schedule/queue/stream trigger, add a dedicated
function:

```yaml
functions:
  processRefunds:
    handler: src/handlers/jobs/processRefunds.handler
    events:
      - schedule: rate(15 minutes)
```

Handler goes in `src/handlers/jobs/`, one file per function. Remember this
project is Serverless Framework **v3** syntax.

## 4. Tests

Next to the router: `src/routes/refunds.test.ts`. Test through the app, not
the internals:

```ts
import { app } from '../app'

it('rejects a non-uuid orderId with 400', async () => {
  const res = await app.request('/refunds', {
    method: 'POST',
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ orderId: 'nope', amountCents: 100, reason: 'other' }),
  })
  expect(res.status).toBe(400)
})
```

Cover at minimum: the happy path, one validation failure, one not-found/conflict case.

## 5. Verify before finishing

```
pnpm typecheck && pnpm lint && pnpm test
```

If a new env var was introduced, confirm it exists in all three places:
`serverless.yml` `provider.environment`, `src/lib/env.ts`, `.env.example`.
