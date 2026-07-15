---
name: api-design
description: HTTP API contract conventions — resource naming and verbs, versioning, the status-code policy, success/list/error response shapes, pagination cursors, idempotency, and contract-first schemas.
---

# API design

The CONTRACT rules — consistency across every endpoint so the frontend
http-client and every consumer can rely on one set of shapes. Owns the
success/list shapes and the conventions; the error shape is owned by
hono-patterns §4 and referenced here as part of the contract.

## 1. Resources & verbs

- Nouns, plural, kebab-case: `/refund-requests`, not `/getRefundRequests`
- Verbs are HTTP methods: `GET` (safe, cacheable), `POST` (create),
  `PUT`/`PATCH` (replace/merge — pick one meaning per repo [adapt]),
  `DELETE`. An action that isn't CRUD → a sub-resource
  (`POST /orders/:id/refunds`) or a documented `POST /orders/:id:cancel`
  RPC-style exception [house policy — adapt], never `GET` with side effects
- Nesting ≤ 1 level (`/orders/:id/items`); deeper → top-level resource with
  a filter. IDs in the path, never in the body for identity

## 2. Versioning

- URL prefix `/v1` [adapt: vs header — pick ONE repo-wide]; version the
  whole API, not per-endpoint
- Additive changes (new optional field, new endpoint) do NOT bump the
  version — consumers tolerate unknown fields (their zod schemas don't
  `.strict()` responses)
- Breaking change = new version; both live during a deprecation window with
  a sunset date communicated [adapt: Deprecation/Sunset headers]. Same
  discipline as eventbridge event versioning

## 3. Status codes — the policy (consistent or consumers guess)

- `200` read/update ok · `201` created (+ `Location` or the entity) ·
  `204` delete/no-body
- `400` malformed/validation (body from zValidator) · `401`
  unauthenticated · `403` authenticated-but-forbidden · `404` not found
  (and cross-tenant probes per api-security §4's 404 policy)
- `409` conflict — the conditional-write/version clash
  (dynamodb §2, rds transactions) · `422` only if the repo distinguishes
  it from 400 [adapt — most don't] · `429` rate limited (+ `Retry-After`,
  which axios honors)
- `5xx` only for genuine server faults (hono-patterns onError) — never for
  a client's bad input

## 4. Response shapes

- **Single resource**: the entity, enveloped-or-bare CONSISTENTLY [adapt:
  bare entity is common; if you envelope, envelope everything]
- **List** — cursor-based, one shape everywhere:

```json
{ "items": [ ... ], "nextCursor": "eyJwayI6..." }
```

  `nextCursor` is opaque + encoded (dynamodb §3 — never raw keys); absent/
  null = last page. Offset/`page` pagination only where a stable total is
  required and the store supports it [adapt]
- **Error**: `{ "error": { "code", "message", "details?" } }` — owned by
  hono-patterns §4; `code` is machine-readable and stable, `message` is
  safe-for-client. The frontend maps `code`, not `message`
- Field naming consistent repo-wide [adapt: camelCase JSON is common];
  dates are ISO-8601 UTC strings (frontend parses with date-fns)

## 5. Request conventions

- **Filtering/sorting** via query params, zod-validated against a WHITELIST
  (`status`, `createdAfter`), `sort=-createdAt` (leading `-` = desc) —
  arbitrary field sorting is an injection/perf hole (api-security)
- **Pagination**: `limit` (capped [adapt: default 20, max 100]) + `cursor`
- **Idempotency**: unsafe-but-retryable POSTs accept `Idempotency-Key`
  header, deduped server-side (lambda-patterns §4, dynamodb conditional
  write) — this is the backend half of axios's "POST never blind-retries"
- Partial responses/field selection only if a real need exists — don't
  build GraphQL-lite speculatively

## 6. Contract-first

- The zod schemas ARE the contract (zod-schemas); request/response types
  derive from them — hand-written API types drift (typescript §5)
- Shared FE+BE contract [adapt: a contracts package exporting the schemas —
  the one clear cross-boundary DRY win, same as event contracts]
- OpenAPI generated from the schemas if the org needs it [adapt:
  @hono/zod-openapi] — generated, never hand-maintained alongside code

## 7. Verify

New endpoint: its list shape, error shape, and status codes match the
policy above BEFORE implementation (review the contract first). Consistency
pass across the service — grep for endpoints returning bespoke list/error
shapes. The frontend consumes it through http-client with no special-casing;
if the frontend needs a per-endpoint workaround, the contract broke a rule.
