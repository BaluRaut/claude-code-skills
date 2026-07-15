---
name: lambda-patterns
description: Lambda fundamentals — init-once/reuse, cold-start hygiene, timeout budgets, retry semantics per trigger, idempotency as a requirement, DLQs.
---

# Lambda patterns

## 1. Init once, reuse across invocations

Module scope = warm-container cache. Clients, schema compilation, config
reads happen OUTSIDE the handler:

```ts
const db = DynamoDBDocument.from(new DynamoDBClient({}))  // module scope
export const handler = async (event) => { /* use db */ }
```

Never cache per-REQUEST data (user, tenant) in module scope — that's a
cross-invocation data leak between users.

## 2. Cold starts — spend where it matters

- Bundle size is the lever you control: modular AWS SDK v3 imports
  (aws-sdk-v3), no accidental heavy deps (check with serverless-v3-config §3)
- Lazy-init rarely-used clients inside the code path that needs them
- Provisioned concurrency only for measured, user-facing latency problems —
  it's a bill, not a default

## 3. Timeout budgets (the chain must add up)

Function timeout > sum of downstream timeouts, or you get zombie work:
set explicit timeouts on every outbound call (SDK client timeouts, fetch
AbortSignal) so the FUNCTION never dies mid-flight with no error handling.
SQS consumers: visibility timeout ≥ 6× function timeout (sqs skill).

## 4. Retry semantics — know your trigger, or double-execute

- API Gateway/HTTP: NO automatic retry — client handles it
- SQS: redelivery until success or maxReceiveCount → DLQ
- EventBridge/async invoke: 2 retries built-in, then DLQ/destination
- Streams: retry until success — a poison record blocks the shard without
  bisect/DLQ config

**Therefore: every non-HTTP handler is idempotent.** Natural idempotency
(deterministic PUT) or an idempotency key check [adapt: conditional write
on the table]. "It probably won't retry" is not a design.

## 5. Failure wiring — no silent async failures

Every async function (queue, schedule, event) has: a DLQ or failure
destination, an alarm on it (logging-observability), and a documented
replay procedure [adapt: redrive]. An empty DLQ nobody watches is the same
as no DLQ.

## 6. Verify

`serverless package` diff for bundle bloat; force one failure in dev and
watch it land in the DLQ + alarm; kill a request mid-flight and confirm
timeouts produce a logged error, not silence.
