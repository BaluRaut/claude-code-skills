---
name: cloudfront
description: CloudFront conventions — SPA and API origins, cache policy by content class, OAC-locked buckets, security headers, invalidation discipline.
---

# CloudFront

## 1. Cache by content class (the whole game)

| Content | Policy |
|---|---|
| Hashed assets (`/assets/*.abc123.js`) | cache ~forever (`max-age=31536000, immutable`) |
| `index.html` | NO cache (`no-cache` / max-age=0) — it's the pointer to the hashes |
| API behaviors (`/api/*`) | caching disabled unless a response is DESIGNED cacheable |
| User/tenant-specific responses | never cached at the edge without keying on auth — one user's data served to another is the worst-case bug |

The frontend deploy skill's "sync + invalidate" works BECAUSE of the
index.html rule — hashed assets never need invalidating.

## 2. Origins

- S3 origin: bucket private + **OAC** (origin access control) — the bucket
  policy allows only this distribution; a public bucket behind CloudFront
  fails review (s3 skill §2)
- API origin [adapt: API Gateway / ALB]: forward the headers/cookies the
  API actually needs via origin request policy — `Authorization` forwarding
  is explicit, not accidental
- SPA routing: 403/404 → `/index.html` with 200 (custom error responses),
  scoped so API paths still return real errors

## 3. Security headers — at the edge, one place

Response headers policy [adapt: managed or custom]: HSTS,
X-Content-Type-Options, frame-ancestors/CSP (frontend-security), and
TLS minimum on the distribution. Headers set here, not per-app.

## 4. Invalidation discipline

- Deploys invalidate `/*` (or `/index.html` + non-hashed paths) — cheap at
  your scale, and "some users on the old version for hours" costs more
- Invalidation is NOT the fix for "we cached user-specific data" — the
  cache key/policy is (§1)

## 5. Verify

After deploy: response headers show the expected `cache-control` +
`x-cache: Hit` on assets and `Miss/Disabled` on API; a hard refresh serves
the new build hash; direct-to-bucket URL returns 403 (OAC working);
security headers present on both HTML and API responses.
