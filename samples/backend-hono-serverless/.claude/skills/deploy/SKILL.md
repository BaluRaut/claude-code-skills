---
name: deploy
description: Deploy this service to AWS with Serverless Framework v3 — pre-flight checks, stage rules, post-deploy verification, and rollback.
---

# Deploy

## Stage rules

- **dev** — deploy freely: `pnpm deploy:dev`
- **staging** — only from an up-to-date `main` branch
- **prod** — NEVER run a prod deploy unless the user explicitly confirmed it
  in this conversation. Ask, showing the diff of what will ship.

## Pre-flight (all must pass — stop and report if any fails)

1. `pnpm typecheck && pnpm lint && pnpm test`
2. `git status` is clean and the branch matches the stage rule above.
3. `npx serverless print --stage <stage>` resolves without error — this catches
   missing SSM params/env config *before* CloudFormation does.

## Deploy

```
npx serverless deploy --stage <stage> --verbose
```

Fast iteration on dev when only handler code changed (no serverless.yml change):

```
npx serverless deploy function -f api --stage dev
```

## Verify

1. `npx serverless info --stage <stage>` — grab the endpoint URL.
2. `curl <endpoint>/health` — expect 200.
3. `npx serverless logs -f api --stage <stage> --startTime 5m` — no new errors.

Report the endpoint URL and verification results to the user.

## Rollback

```
npx serverless deploy list --stage <stage>   # find the previous timestamp
npx serverless rollback -t <timestamp> --stage <stage>
```

Roll back first, investigate second — don't debug a broken stage live.
