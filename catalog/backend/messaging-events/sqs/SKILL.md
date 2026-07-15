---
name: sqs
description: SQS consumer conventions — partial batch failure, visibility vs function timeout, DLQ + redrive, idempotent processing, FIFO only when ordering is real.
---

# SQS

## 1. Consumer wiring (serverless.yml)

```yaml
functions:
  processOrders:
    handler: src/handlers/jobs/processOrders.handler
    timeout: 30
    events:
      - sqs:
          arn: !GetAtt OrdersQueue.Arn
          batchSize: 10                      # [adapt to workload]
          functionResponseType: ReportBatchItemFailures   # non-negotiable
```

- **`ReportBatchItemFailures` always** — without it, one bad message
  re-delivers the whole batch: duplicates for the 9 that succeeded
- Queue visibility timeout ≥ 6× function timeout (AWS guidance) — too low
  = concurrent double-processing of in-flight messages

## 2. Handler shape — per-message isolation

```ts
export const handler = async (event: SQSEvent): Promise<SQSBatchResponse> => {
  const failures: SQSBatchItemFailure[] = []
  for (const record of event.Records) {
    try { await processOne(parse(record)) }        // zod-parse the body
    catch (e) { log.error({ e, messageId: record.messageId }); 
                failures.push({ itemIdentifier: record.messageId }) }
  }
  return { batchItemFailures: failures }
}
```

One message's failure never fails its neighbors. `processOne` is a plain
service function (hono-patterns §1) — testable without SQS event JSON.

## 3. Idempotency — duplicates are a FEATURE of SQS

At-least-once delivery means `processOne` runs twice for the same logical
message sometimes. Conditional write on an idempotency key [adapt:
DynamoDB attribute_not_exists with TTL] or naturally idempotent operations.
This is lambda-patterns' rule made concrete.

## 4. DLQ — with a plan, not just an ARN

- Every queue has a DLQ, `maxReceiveCount` [adapt: 3–5], an ALARM on DLQ
  depth > 0 (logging-observability), and a documented redrive step
  (console redrive or CLI) in the runbook
- A DLQ message is a bug report: debug-prod-error it, fix, THEN redrive —
  redriving into the same bug is a loop

## 5. Choices to justify in the PR

- FIFO only when ordering/exactly-once semantics are a real requirement —
  it costs throughput and adds MessageGroupId design; "just in case" FIFO
  fails review
- Producer side: messages carry a zod-schema'd body with a `type` and
  `version` (same contract discipline as eventbridge)
- Long-running work >15min doesn't belong on Lambda consumers [adapt:
  Step Functions / containers]

## 6. Verify

Test: batch of 3 with the middle one failing → response lists exactly one
failure. Dev: send a poison message, watch it hit the DLQ after
maxReceiveCount and fire the alarm; redrive it after a fix.
