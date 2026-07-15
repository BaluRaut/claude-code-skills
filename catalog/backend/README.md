# Backend skill catalog (Node serverless on AWS)

Same rules as the frontend catalog: **nothing installs verbatim** — adapt
`[...]` placeholders to the repo's real stack, install 5–8 per service repo
(only for infrastructure the service ACTUALLY uses — an `sqs` skill in a
repo with no queues teaches Claude to invent queues), prune monthly.

Stack assumed: Node 20 + Hono + Serverless Framework v3 + AWS SDK v3.
The `new-endpoint` and `deploy` task skills live in
[../../samples/backend-hono-serverless/](../../samples/backend-hono-serverless/).

## Service Patterns — [service-patterns/](service-patterns/)

| Skill | Covers |
|---|---|
| hono-patterns | routers, middleware order, zod validation, error taxonomy |
| hono-middleware | writing middleware: await-next contract, short-circuit, testing |
| api-design | REST naming, versioning, status-code policy, list/error shapes, cursors |
| serverless-v3-config | serverless.yml discipline, per-function IAM, packaging |
| lambda-patterns | cold starts, timeouts, retries semantics, idempotency |
| aws-sdk-v3 | client reuse, modular imports, pagination, retry config |

## Data & Storage — [data-storage/](data-storage/)

| Skill | Covers |
|---|---|
| dynamodb | access patterns first, keys/GSIs, pagination, TTL, hot keys |
| rds-postgres | RDS Proxy from Lambda, migrations, pooling, VPC reality |
| s3 | presigned URLs, encryption, lifecycle, event notifications |

## Messaging & Events — [messaging-events/](messaging-events/)

| Skill | Covers |
|---|---|
| sqs | partial batch failure, visibility vs timeout, DLQ, idempotent consumers |
| eventbridge | event contracts/versioning, rules, DLQs, SQS-vs-EventBridge choice |

## Security & Config — [security-config/](security-config/)

| Skill | Covers |
|---|---|
| api-security | authN/Z middleware, least-privilege IAM, secrets never in code |
| secrets-config | Secrets Manager vs Parameter Store, caching, rotation |

## Infra & Delivery — [infra-delivery/](infra-delivery/)

| Skill | Covers |
|---|---|
| cloudfront | cache policies, invalidation, security headers, origins |
| vpc-networking | when Lambda needs a VPC, endpoints, NAT cost, EC2 via SSM |
| containers-eks-ecr | image hygiene, ECR lifecycle, probes/resources, deploy flow |

## Observability — [observability/](observability/)

| Skill | Covers |
|---|---|
| logging-observability | pino + correlation IDs, metrics/alarms, log hygiene |
| debug-prod-error | task: the triage runbook — logs → trace → cause → rollback/fix |
| sentry-node | capture-once in onError, flush-before-freeze, source maps |
| datadog-apm | extension layer, unified tagging, RUM trace correlation, log injection |

## Code Quality — [code-quality/](code-quality/)

| Skill | Covers |
|---|---|
| code-design-solid | handler/service/repo layering, size limits, DRY across services |

## Testing — [testing/](testing/)

| Skill | Covers |
|---|---|
| node-testing | runner conventions, aws-sdk-client-mock, route tests, local infra |
