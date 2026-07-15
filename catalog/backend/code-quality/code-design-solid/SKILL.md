---
name: code-design-solid
description: Design standards for Node services — handler/service/repository layering as SRP, size and cognitive-load limits, pure testable business logic, DRY across services with the rule of three.
---

# Code design (backend)

## 1. The layering IS the design (SRP made mechanical)

```
handlers/   → translate transport → service call (HTTP, SQS event, cron)
services/   → business rules — plain functions, no Hono/Lambda/AWS types
db|repos/   → data access (dynamodb/rds-postgres conventions)
lib/        → cross-cutting seams (logger, monitoring, aws wrappers)
```

- A handler with business logic, or a service importing `hono` or reading
  `process.env`, is the review catch — each layer has one reason to change
- The payoff is node-testing §1: services test without AWS or event JSON;
  the SAME service function serves the HTTP route and the queue consumer
  (sqs §2) — transport is a detail

## 2. Size & cognitive-load limits [numbers — adapt]

- File ~300 lines; function ~40, nesting ≤ 3 (early returns/guard
  clauses), params ≤ 3 then an options object
- A service function needing 6 mocks to test is telling you it does 3
  jobs — split it (the test-pain signal beats any line count)
- Compound conditions get NAMED: `const canRefund = ...` — same rule as
  the frontend skill

## 3. SOLID, translated to services

- **S** — §1's layers; plus one handler per trigger concern
  (serverless-v3-config §1)
- **O** — new event/message types extend a handler REGISTRY
  (`handlers[event.type]`), not another branch in a growing switch;
  unknown types fail loudly (eventbridge §2)
- **L** — implementations of a seam honor its contract: a repo's fake
  (tests) and real (dynamo) versions throw the same error types
- **I** — narrow signatures: `processRefund({ orderId, amountCents })`,
  not `(event: SQSEvent)` passed three layers deep — transport types stop
  at the handler
- **D** — services depend on the wrappers (aws-sdk-v3 §1, secrets-config,
  logger), injected or imported as seams — never `new S3Client` inline in
  business code

## 4. DRY across services — the distributed rule of three

- Within a service: same as frontend — third occurrence + stable shape =
  extract; named modules, no `utils.ts` dumping ground
- **Across services**: duplication is often CORRECT — a shared lib couples
  deploy cycles and teams. Share only what's stable and truly one piece of
  knowledge [adapt: contracts package for event/API schemas — the one
  clear win (eventbridge §2)]; copy-paste-and-own the rest until it's
  proven stable
- Knowledge that must never fork: error body shape (hono-patterns §4),
  event contracts, key builders (dynamodb §1), correlation-id convention
  — these ARE the system's interfaces
