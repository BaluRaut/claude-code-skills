---
name: rds-postgres
description: Postgres from Lambda — RDS Proxy and connection discipline, migrations as artifacts, query conventions, VPC reality, and read-path safety.
---

# RDS / Postgres from Lambda

## 1. Connections — the thing that takes prod down

Lambda scale-out × direct connections = exhausted Postgres. Rules:

- **RDS Proxy** (or [adapt: Aurora Data API]) between Lambdas and the DB —
  direct connections from Lambda fail review
- One client per container at module scope, pool size 1
  (`max: 1` — concurrency comes from Lambda instances, not in-function
  pools) [driver — adapt: pg / postgres.js / an ORM]
- Auth via IAM auth or Secrets Manager (secrets-config) — password in env
  plaintext fails review
- Statement timeout set (`statement_timeout` [adapt value]) — a stuck query
  must die before the function does (lambda-patterns §3)

## 2. Migrations — versioned artifacts, never manual

- Tool: [adapt — node-pg-migrate / drizzle-kit / prisma migrate]; files
  committed, sequential, reviewed like code
- Run as an explicit deploy step [adapt: migration function invoked by
  pipeline], never at handler cold start (N containers racing migrations)
- Backwards-compatible by default: expand → deploy code → contract in a
  LATER release. Dropping/renaming a column the running version still
  reads is the classic self-inflicted outage
- Destructive migrations (drop/type change): explicit human confirmation,
  same rule as prod deploys

## 3. Query conventions

- Parameterized queries ONLY — string interpolation into SQL is an
  automatic review block (api-security)
- Queries live in [src/db/ repositories — adapt], typed results at the
  boundary (zod or the ORM's types); handlers never hold SQL
- Every list query has LIMIT + keyset/cursor pagination; transactions for
  multi-statement invariants — and kept SHORT (no external calls inside a
  transaction)
- N+1 from loops of queries: batch with `WHERE id = ANY($1)` — new
  endpoint review checks this

## 4. VPC reality (pairs with vpc-networking)

DB in private subnets → the Lambda joins the VPC → it loses direct internet
unless NAT exists, and needs VPC endpoints for AWS APIs it calls. Check
what ELSE the function talks to BEFORE moving it into the VPC — "added RDS,
broke S3 calls" is the classic.

## 5. Verify

Migration applies + rolls back on a scratch DB; integration tests against
[docker compose postgres — adapt]; under parallel invocation in dev,
connection count on the proxy stays flat.
