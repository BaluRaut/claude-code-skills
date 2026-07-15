---
name: node-testing
description: Testing Node services — runner conventions, the three test layers, aws-sdk-client-mock, Hono route tests, local infra via docker, event-fixture reuse.
---

# Node service testing

Runner: [vitest / jest — adapt to the repo; one runner, and the matching
frontend skill's mocking/timer rules apply]. Layers, cheapest first:

## 1. Unit — services and pure logic (most tests live here)

- Service functions (hono-patterns §1 keeps them plain) tested directly;
  derived from acceptance criteria — the write-unit-tests discipline
  applies verbatim on the backend
- AWS at the boundary via `aws-sdk-client-mock` (aws-sdk-v3 §6):

```ts
const dbMock = mockClient(DynamoDBDocumentClient)
dbMock.on(UpdateCommand).rejects(new ConditionalCheckFailedException({...}))
// assert the 409 path, not just happy paths
```

- Mock reset in beforeEach; assert CALLS for contract-critical writes
  (condition expressions, idempotency keys — sqs §3)

## 2. Route tests — through the app, no server

`app.request()` against the real Hono app (hono-patterns §6): validation
400s, authZ 401/403/IDOR-404 (api-security §6), error-shape 500. Auth via
[adapt: test JWT helper / injected user context].

## 3. Integration — real local infra where the risk is

- dynamodb-local / postgres via docker compose [adapt: `docker compose up
  -d` before the integration suite; separate script `test:integration`]
- What earns integration coverage: each DynamoDB access pattern
  (dynamodb §5), migrations apply+rollback (rds-postgres §5), pagination
  across page boundaries — the things client-mocks can't prove
- CI runs both suites [adapt]; integration data is per-test-isolated
  (parallel-safety, same rule as playwright-e2e fixtures)

## 4. Event handlers — fixtures from production shapes

- SQS/EventBridge handler tests feed REAL event JSON shapes (grab from
  logs/DLQ once, commit as fixtures) — hand-typed fake events drift
- Fixtures parse against the contract schemas (eventbridge §6) so contract
  changes fail the fixture loudly — the msw-mocking drift-alarm trick
- Cover: partial batch failure (sqs §6), duplicate delivery (idempotency),
  unknown event version rejection

## 5. What NOT to do

- No localstack-everything by default — mock units, docker the stateful
  stores, verify the rest in a dev stage deploy [house line — adapt]
- Don't test AWS itself ("S3 stores files") — test YOUR keys, conditions,
  and error handling
- No `--forceExit`/open-handle suppression — an unclosed client is a bug

## 6. Verify

`pnpm test` (unit+route) fast enough to run every save; `pnpm
test:integration` green with docker up; the repo verify command runs both
before a PR (common-rules §4).
