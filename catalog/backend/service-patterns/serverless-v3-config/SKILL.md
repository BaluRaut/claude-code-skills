---
name: serverless-v3-config
description: serverless.yml discipline for Serverless Framework v3 — stages, per-function IAM, esbuild packaging, resources, and the config review checklist.
---

# Serverless Framework v3 config

Version pinned: **v3.x (3.40)** — no v4-only syntax, don't suggest
upgrading in a feature PR. Plugins: `serverless-esbuild`,
`serverless-offline` [adapt].

## 1. Layout rules

- `provider`: runtime nodejs20.x, region/stage from CLI (`--stage`), shared
  env in `provider.environment` — values via `${ssm:...}` / `${env:...}`,
  never literals for anything secret (secrets-config)
- One function per INDEPENDENT concern: the catch-all `api` + one function
  per queue/schedule/stream. Don't split HTTP routes into per-route
  functions without a scaling/IAM reason
- `resources:` (CloudFormation) for infra the service OWNS (its table, its
  queue). Shared infra lives in [the infra repo — adapt] and is referenced
  by ARN/SSM — a service creating another team's bucket is the smell

## 2. IAM — per function, least privilege

```yaml
functions:
  processOrders:
    handler: src/handlers/jobs/processOrders.handler
    iamRoleStatements:            # via serverless-iam-roles-per-function [adapt]
      - Effect: Allow
        Action: [dynamodb:GetItem, dynamodb:UpdateItem]
        Resource: !GetAtt OrdersTable.Arn
```

`Action: ['dynamodb:*']` or `Resource: '*'` fails review — name the calls
the code makes. New SDK call = matching IAM statement in the same PR.

## 3. Packaging (esbuild)

- `individually: true` for mixed workloads; externals only for layers/
  binaries; runtime deps in `dependencies` NOT `devDependencies` (silent
  bundle omission otherwise)
- Check bundle size on new heavy deps: `serverless package` and look —
  cold start pays for every MB (lambda-patterns)

## 4. Function defaults — decide, don't inherit

`memorySize` and `timeout` set explicitly per function class [adapt house
defaults: api ~512MB/15s, jobs per workload]. HTTP behind API Gateway
caps ~29s anyway — a 900s timeout on the api function is a config lie.

## 5. Review checklist for any serverless.yml diff

1. `npx serverless print --stage dev` resolves (catches bad refs pre-deploy)
2. New env var → env.ts schema + .env.example too
3. IAM diff matches the code diff
4. Event source config sanity: SQS `batchSize`/`functionResponseTypes`,
   schedule expressions in UTC
5. Nothing destructive to stateful resources (DeletionPolicy: Retain on
   tables/buckets [adapt])
