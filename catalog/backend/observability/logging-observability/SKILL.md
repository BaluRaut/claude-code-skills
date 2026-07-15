---
name: logging-observability
description: Structured logging with pino, correlation IDs end-to-end, log levels that mean something, redaction, and the metric/alarm-per-failure-mode rule.
---

# Logging & observability

## 1. Structured logs, one logger

- pino via [src/lib/logger.ts — adapt] — `console.log` fails review
  (it produces unqueryable strings)
- JSON fields, not interpolated prose:
  `log.info({ orderId, durationMs }, 'order processed')` — CloudWatch Logs
  Insights queries fields, not regexes
- Base context bound once per invocation: `requestId` (from Hono middleware
  / Lambda context), `service`, `stage`, `version`

## 2. Correlation — one ID through the whole story

- HTTP: accept inbound `x-request-id` or generate; return it in responses;
  the frontend's http-client/monitoring carries the same ID — a user bug
  report joins its backend logs (error-monitoring's context rule, mirrored)
- Async: the ID travels IN the message/event body (sqs/eventbridge
  contracts include it); consumers bind it to their logger — a five-hop
  flow is one Insights query

## 3. Levels with meaning (or dashboards lie)

- `error`: unexpected, actionable, alarm-worthy — 4xx client mistakes are
  NOT errors (api-security §4)
- `warn`: degraded-but-handled (retry succeeded, fallback used)
- `info`: business events, one per invocation outcome — not per step
- `debug`: off in prod by default [adapt: env-controlled]
- Log-once rule: report an error at the layer that HANDLES it — the same
  exception logged at three layers makes error rates fiction

## 4. Redaction — configured, not remembered

pino `redact` paths for auth headers, tokens, passwords, PII fields
[adapt list] — enforced in the logger config so nobody has to remember
per-call (secrets-config §4, api-security §4).

## 5. Metrics & alarms — one per failure mode

Every function answers: "how do I know it's broken before a user tells
me?" Minimum set [adapt: CloudWatch alarms / Datadog — same tagging as
datadog-rum if both]:

- API: 5xx rate, p99 latency
- Queues: DLQ depth > 0, oldest-message age
- Jobs: failed-invocation count, "didn't run" (missing-invocation alarm on
  schedules)
- Business-critical counts via EMF/custom metrics [adapt] where silence
  is itself the failure

Alarms page a CHANNEL someone owns [adapt]; an alarm nobody receives is
documentation.

## 6. Verify

Insights query by one requestId returns the full story across functions;
forced error appears ONCE at error level with context; redaction test:
log a fake token field, confirm it's masked; each new failure mode in the
PR names its alarm.
