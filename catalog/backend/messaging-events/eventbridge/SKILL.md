---
name: eventbridge
description: EventBridge conventions — event contracts and versioning, publishing rules, rule targets buffered through SQS, DLQs, and when EventBridge vs SQS.
---

# EventBridge

## 1. SQS or EventBridge? (decide per case, in the PR)

- **SQS**: one known consumer doing work — a command ("process this")
- **EventBridge**: a FACT other services may care about — "OrderPlaced" —
  with 0..N consumers you don't want to know about
- Publishing an event AND directly calling the consumer is the anti-pattern
  — pick one contract

## 2. Event contract — schema'd, versioned, factual

```ts
// [src/events/order-placed.ts — adapt; shared contracts package if cross-service]
export const orderPlacedV1 = z.object({
  version: z.literal(1),
  orderId: z.string().uuid(),
  customerId: z.string().uuid(),
  totalCents: z.number().int(),
  occurredAt: z.string().datetime(),
})
```

- `source: 'acme.orders'`, `detail-type: 'OrderPlaced'` — past-tense fact,
  stable naming [adapt house registry of sources/types]
- Carry ids + facts, not full mutable entities — consumers fetch fresh
  state if they need more (fat events go stale in flight)
- Breaking change = new `version`, publish both during migration; consumers
  zod-parse and REJECT unknown versions loudly, not silently

## 3. Publishing

- One publisher wrapper [src/lib/events.ts — adapt] — validates against the
  schema before `PutEvents`, adds correlation id (logging-observability)
- `PutEvents` is batched (10 max) and can PARTIALLY fail — check
  `FailedEntryCount`, don't fire-and-forget
- Publish AFTER the state change commits; for must-not-miss events use
  [adapt: transactional outbox / DynamoDB streams] — "saved but event lost"
  is the consistency bug

## 4. Consuming — buffer through SQS

Rule → **SQS → Lambda** (all the sqs skill's machinery: batching, DLQ,
redrive, backpressure), not rule → Lambda directly, for anything that does
real work. Every rule/target gets a DLQ + alarm; consumers are idempotent.

## 5. Rules & schedules

- Rules in the CONSUMER's serverless.yml (consumer owns what it reacts to);
  pattern-match on `source` + `detail-type`, not deep detail matching
- Scheduled jobs: `schedule:` expressions are UTC — comment the intended
  local time; prefer EventBridge Scheduler for new timezone-aware cron
  [adapt]

## 6. Verify

Contract test: publisher fixture parses with the consumer's schema (same
drift-alarm trick as msw-mocking fixtures). Dev: publish a test event,
confirm delivery through rule → SQS → handler, and an unknown-version
event lands in the failure path, not silence.
