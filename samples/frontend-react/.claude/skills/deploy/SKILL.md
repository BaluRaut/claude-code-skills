---
name: deploy
description: Build and deploy this React app — pre-flight checks, stage rules, upload, cache invalidation, and verification.
---

# Deploy

> Placeholder infra: this template assumes S3 + CloudFront. Replace the upload
> and invalidation steps with your real target (Amplify, Vercel, internal CDN…).

## Stage rules

- **dev** — deploy freely
- **staging** — only from an up-to-date `main`
- **prod** — NEVER deploy unless the user explicitly confirmed it in this
  conversation. Ask first, stating what will ship.

## Pre-flight (all must pass — stop and report if any fails)

1. `pnpm typecheck && pnpm lint && pnpm test`
2. `pnpm build` — the production build is stricter than dev; it must succeed.
3. `git status` clean, branch matches the stage rule.
4. Confirm the stage's env file exists (`.env.staging`, `.env.production`) and
   contains every `VITE_` var used in `src/lib/env.ts`.

## Deploy

```bash
pnpm build --mode <stage>
aws s3 sync dist/ s3://acme-portal-<stage>/ --delete
aws cloudfront create-invalidation --distribution-id <DIST_ID> --paths "/*"
```

`--delete` removes stale hashed bundles; the invalidation is required or users
keep the old `index.html`.

## Verify

1. Open the stage URL — app loads, no console errors.
2. Hard-refresh once (cache) and check the build hash/version in the footer or
   `/version.json` matches what was just shipped.
3. Exercise one core flow (login → main page renders data).

Report the URL and verification results to the user.

## Rollback

Re-deploy the previous git tag/commit the same way — builds are reproducible
from source. Roll back first, investigate second.
