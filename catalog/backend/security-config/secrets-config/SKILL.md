---
name: secrets-config
description: Secrets Manager vs Parameter Store — which for what, resolution at deploy vs runtime, caching, rotation-tolerant code, and the no-plaintext rules.
---

# Secrets & configuration

## 1. Which store for what

- **Parameter Store (SSM)**: non-secret config (URLs, flags, table names,
  ARNs) + modest secrets as SecureString [house line — adapt]
- **Secrets Manager**: credentials needing ROTATION (DB passwords,
  third-party API keys) — rotation is the feature you pay for
- Naming convention: `/[service]/[stage]/[key]` → `/orders/prod/db-url`
  [adapt] — per-service prefixes enable per-function IAM scoping

## 2. Deploy-time vs runtime resolution (know the difference)

- `${ssm:/orders/dev/queue-url}` in serverless.yml = resolved AT DEPLOY,
  baked into the function env. Fine for config; for SecureStrings it lands
  DECRYPTED in the Lambda env — visible in the console. House rule
  [adapt]: real secrets are fetched at RUNTIME, not baked
- Runtime fetch: module-scope cached (lambda-patterns §1) with TTL [adapt:
  ~5–15 min or the Parameters/Secrets Lambda Extension] — per-request
  fetches add latency and throttle at scale

## 3. Rotation-tolerant code

On auth failure with a cached credential: refresh the secret once and
retry once — that's the rotation window, not an outage. DB credentials:
RDS Proxy + Secrets Manager handles this for you (rds-postgres §1) —
prefer that over hand-rolled rotation handling.

## 4. The no-plaintext rules (reviewed on every diff)

1. No secret values in: code, serverless.yml literals, .env committed
   files, CI variables that echo, logs (logging-observability redaction)
2. `.env.example` lists NAMES with fake values
3. env.ts distinguishes config (validated at startup) from secret HANDLES
   (name/ARN validated; value fetched at runtime)
4. A leaked/committed secret = rotate immediately, THEN clean history —
   rotation is the fix; deletion is cosmetics
5. IAM: functions read their own prefix only
   (`ssm:GetParameter` on `/orders/${stage}/*`) — cross-service secret
   reads are a design smell

## 5. Verify

New secret: appears in the store with the naming convention, function IAM
scoped to its prefix, `serverless print` shows no decrypted value baked
where the house rule forbids it, and a rotation drill in dev (rotate,
invoke, no error after one retry).
