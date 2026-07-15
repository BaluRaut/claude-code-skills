---
name: api-security
description: API security for Hono services — authN/Z middleware placement, input validation as security, tenant isolation, least-privilege IAM, safe errors and logs.
---

# API security

Pairs with frontend-security: the frontend hides buttons; THIS layer is the
actual boundary — every endpoint authorizes regardless of what the UI shows.

## 1. AuthN — verified at the edge, once

- JWT/session verification in middleware [adapt: JWKS-cached verifier /
  API GW authorizer], applied by path scope (hono-patterns §2) — a route
  opting OUT of auth is explicit and reviewed (`/health`, webhooks)
- Webhooks: signature verification (HMAC per vendor) BEFORE parsing;
  reject on clock skew/replay [adapt per integration]
- Internal service-to-service: IAM/SigV4 or scoped tokens [adapt] — never
  a shared static API key in env

## 2. AuthZ — object-level, not just route-level

The classic hole is IDOR, not missing login: `GET /orders/:id` must check
the order BELONGS to the caller/tenant, not just that a token exists.

- Tenant/ownership predicate in the SERVICE layer on every read/write
  [adapt: tenantId in every key/query — see dynamodb key design]
- Roles/permissions checked with one helper (mirror of frontend
  `can()`) — scattered `user.role === 'admin'` fails review here too

## 3. Input is hostile — validation IS security

- zValidator on every input (hono-patterns §3): unknown fields stripped,
  sizes bounded (string max, array max, pagination limit caps)
- SQL parameterized only (rds-postgres); DynamoDB keys built by builders,
  never from raw client strings; no user input into shell/eval/paths
- File uploads: server-generated keys, content-type pinned (s3 skill)

## 4. Don't leak

- Error bodies: safe `code`/`message` only — no stack traces, no SQL, no
  internal ARNs (hono-patterns onError owns this)
- Logs: no tokens, passwords, full PII (logging-observability redaction);
  404 vs 403 policy decided consciously [adapt: 404 for cross-tenant
  probing]
- CORS: explicit origins from env — `*` with credentials fails review

## 5. Least privilege everywhere

Per-function IAM matching actual calls (serverless-v3-config §2); secrets
via secrets-config, never committed or in plaintext env; public endpoints
rate-limited [adapt: WAF / API GW throttling] with a stated budget.

## 6. The OWASP extras (checked when the pattern appears)

- **SSRF**: any user-influenced URL fetched server-side (webhooks-out,
  importers, image proxies) → allowlist of hosts/schemes, resolve-and-block
  private ranges + cloud metadata (169.254.169.254), no redirects-follow
  into private space
- **Mass assignment**: never spread a request body into a DB write —
  zod schemas with explicit fields (`.strict()` [adapt]) ARE the
  allowlist; `{ ...body }` into an update is the review catch
- **CSRF**: cookie-based auth → SameSite + CSRF token [adapt]; pure
  bearer-token APIs are exempt — know which one each endpoint is
- **Password handling** (only if the service stores credentials [adapt:
  most use the IdP — then this section is N/A]): argon2id/bcrypt with
  vetted libraries, never hand-rolled, never reversible, never logged
- **Audit logging**: security-relevant events (login/logout, permission
  changes, data exports, admin actions) logged with actor + target +
  requestId (logging-observability) — queryable answer to "who did what"

## 7. Verify

Per new endpoint: 401 no token, 403/404 wrong tenant (IDOR test — the one
that matters), 400 oversized/malformed input, error body leaks nothing.
These four tests ship in the same PR (node-testing).
