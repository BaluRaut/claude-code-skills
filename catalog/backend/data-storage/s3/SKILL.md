---
name: s3
description: S3 conventions — presigned URLs for user data, key design, encryption/access defaults, lifecycle rules, event notifications done safely.
---

# S3

## 1. User files move via presigned URLs — not through Lambda

Upload/download streams through the 6MB-payload, timeout-bound Lambda fail
at scale. Pattern:

1. Endpoint authorizes (api-security), generates presigned PUT/GET
   (`@aws-sdk/s3-request-presigner`), short expiry [adapt: minutes]
2. Presigned PUT pins `ContentType` and length limits [adapt: POST policy
   for strict size caps]; key is SERVER-generated — never a client-chosen
   path (`../` traversal into other tenants' prefixes)
3. Client uploads directly; confirmation via S3 event or a confirm call

## 2. Bucket defaults (every bucket this service owns)

- Block Public Access ON — public content goes through CloudFront + OAC
  (cloudfront skill), never a public bucket
- Encryption on (SSE-S3 baseline; SSE-KMS when key control is required
  [adapt]) — enforced in the bucket config, not per-call
- Versioning on buckets where overwrite/delete is a real risk [adapt]
- Bucket names/ARNs from env/SSM — hardcoded bucket names fail review

## 3. Key design & lifecycle

- Keys are structured, documented, and built in one module [src/lib/
  storage.ts — adapt]: `tenant/{tenantId}/orders/{orderId}/invoice.pdf` —
  tenant prefix FIRST enables per-tenant IAM conditions and cleanup
- Lifecycle rules for anything with an expiry: temp uploads (abort
  incomplete multipart after 1 day — it silently bills otherwise), exports,
  logs → IA/expire [adapt] — deletion-by-lifecycle beats deletion-by-cron

## 4. Event notifications

- S3 → [adapt: SQS (preferred — batching, DLQ, redrive) / EventBridge] →
  Lambda; direct S3→Lambda only for trivial cases
- Consumers are idempotent (duplicate events are documented behavior) and
  tolerate the key being gone by processing time
- The classic loop: trigger writes to the SAME prefix it listens on —
  filter prefixes so output doesn't re-trigger input

## 5. Verify

Presigned flow exercised end-to-end in dev (upload, wrong content-type
rejected, expiry enforced); `aws s3api get-public-access-block` on new
buckets; force a duplicate event and confirm idempotency.
