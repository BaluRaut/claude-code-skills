---
name: dynamodb
description: DynamoDB conventions — access patterns before schema, key design and GSIs, conditional writes, real pagination, TTL, and the query-don't-scan rule.
---

# DynamoDB

## 1. Access patterns FIRST — written down, then keys

A table change PR starts with the list: "get order by id", "list orders by
customer newest-first", "find pending orders older than X". Keys/GSIs exist
to serve THAT list [table design doc — adapt location]. Adding an access
pattern the keys can't serve = design discussion, not a Scan.

- [House model — adapt: single-table (`PK`/`SK` generic names + entity
  prefixes `ORDER#123`) or table-per-entity. Follow what the repo does]
- Key builders in one module [src/db/keys.ts — adapt] — handlers never
  concatenate `ORDER#${id}` inline

## 2. The non-negotiables

- **Query, never Scan** in request paths. A Scan needs a comment justifying
  it and lives in a job, not an endpoint
- **Writes that assume state use conditions**:
  `ConditionExpression: 'attribute_not_exists(PK)'` for create,
  version/status checks for transitions — last-writer-wins is a silent
  data-loss bug. `ConditionalCheckFailedException` → 409, expected flow
- Multi-item invariants: `TransactWriteItems`, not sequential writes with
  hope
- GSIs are eventually consistent — read-your-own-write goes through the
  base table

## 3. Pagination — real, cursor-based

`Limit` + `LastEvaluatedKey` encoded (base64) as the API cursor
(new-endpoint's list shape). Never fetch-all-then-slice; never expose raw
keys as the cursor without encoding [adapt: sign/encrypt if keys are
sensitive].

## 4. Operational hygiene

- TTL attribute for anything with a natural expiry (sessions, idempotency
  keys) — epoch SECONDS, and code tolerates already-expired-not-yet-deleted
- Hot partition check on new write paths: high-volume writes sharing one PK
  (a global counter, "today" as key) need write sharding [adapt]
- Batch ops (`BatchWrite/BatchGet`): handle `UnprocessedItems` with backoff
  — the SDK does NOT retry those for you
- Streams consumers: idempotent, per lambda-patterns retry rules

## 5. Verify

Tests against dynamodb-local (node-testing): each access pattern has a
test; conditional-write conflict path covered; pagination test crosses a
page boundary (not just page one).
