---
name: aws-sdk-v3
description: AWS SDK v3 conventions — module-scope client reuse, modular imports, paginators, explicit timeouts/retries, and the client-mock testing seam.
---

# AWS SDK v3

## 1. Clients — one per service, module scope, wrapped

- Create once at module scope (lambda-patterns §1), region from the
  environment, never per-request
- Feature code imports thin wrappers [src/lib/aws/ — adapt: `db.ts`,
  `storage.ts`, `queue.ts`], not raw clients scattered everywhere — the
  wrapper is the testing seam and the place config lives once
- DynamoDB: `DynamoDBDocument.from(client)` — nobody hand-marshalls
  AttributeValues in feature code

## 2. Imports — modular, always

```ts
import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3'
```

One `@aws-sdk/client-*` package per service used. Importing the whole SDK
or v2 (`aws-sdk`) in new code fails review — bundle size is cold-start time.

## 3. Timeouts & retries — configured, not defaulted

```ts
new S3Client({
  requestHandler: { requestTimeout: 3_000, connectionTimeout: 1_000 }, // [adapt]
  maxAttempts: 3,
})
```

- Defaults allow multi-second hangs — set explicit timeouts fitting the
  function's budget (lambda-patterns §3)
- SDK retries + your retries + trigger retries multiply — the SDK retries
  transport errors; do NOT wrap SDK calls in your own retry loop

## 4. Pagination — paginators, not manual tokens

```ts
for await (const page of paginateQuery({ client: doc }, params)) { ... }
```

Manual `LastEvaluatedKey`/`NextToken` loops are where "we only got page
one" bugs live. And an unbounded full scan/list in a Lambda is a design
smell — put a limit or move it to a job.

## 5. Errors

Catch SDK errors by `error.name` (`ConditionalCheckFailedException`,
`NoSuchKey`) — never by matching message strings. Expected conditions
(conditional write conflict) are FLOW, handled locally; everything else
propagates to the handler's error path with the request id.

## 6. Testing

`aws-sdk-client-mock` against the wrapper's client (node-testing):

```ts
const s3Mock = mockClient(S3Client)
s3Mock.on(GetObjectCommand, { Key: 'x' }).resolves({ ... })
```

Reset in beforeEach. Never stub global fetch or intercept HTTP to fake AWS.
