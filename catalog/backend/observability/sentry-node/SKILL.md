---
name: sentry-node
description: Sentry on Lambda — wrapper module, capture-once in onError, the flush-before-freeze rule, esbuild source maps, PII scrubbing, one error owner with Datadog.
---

# Sentry (Node/Lambda)

The backend mirror of the frontend error-monitoring skill — same wrapper
rule, same one-owner decision.

## 0. One error owner (same table as datadog-apm §0)

If Datadog also runs here: ONE tool owns error alerting. [House split —
adapt: Sentry = exceptions/alerting, Datadog = APM/metrics/traces.]
Double-capturing gives two wrong error rates and two bills.

## 1. Setup — wrapper + handler wrap

- Features import [src/lib/monitoring.ts — adapt], never `@sentry/*`
  directly
- Init at module scope: `environment` (stage), `release` (git SHA — same
  value serverless.yml/CI injects), `tracesSampleRate` low or 0 if Datadog
  owns tracing [adapt]
- Wrap handlers via [adapt: `Sentry.wrapHandler` from @sentry/aws-serverless
  / the serverless plugin] — it captures unhandled rejections AND handles
  flush (§3)

## 2. Capture once, with the story

- HTTP: capture in Hono's `onError` ONLY (hono-patterns §4) — handlers and
  services never capture-and-rethrow (log-once rule,
  logging-observability §3)
- Queue/event handlers: capture per failed item next to the
  batchItemFailures push (sqs §2)
- Context on every event: `requestId` (the SAME correlation id pino logs —
  a Sentry issue must join its CloudWatch story), function name, event
  source; tags for tenant [adapt]
- 4xx expected responses are NOT captured (api-security §4) —
  `HTTPException` < 500 filtered in `beforeSend`

## 3. The Lambda-specific rule: flush before freeze

Lambda freezes the container the moment the handler returns — buffered
events die unsent. `wrapHandler` flushes for you; any custom entry point
(streams, cron without the wrapper) ends with
`await Sentry.flush(2000)`. Missing flush = "Sentry works locally, silent
in prod" — that exact mystery.

## 4. Source maps & scrubbing

- esbuild `sourcemap: true` + upload via [adapt: sentry-cli in CI /
  serverless-sentry plugin] keyed to the same `release` — verify one prod
  stack trace shows TS file/line before trusting it
- `beforeSend` scrubs: auth headers, tokens, request bodies with PII
  [adapt list — same fields pino redacts]. Sentry defaults are not your
  policy

## 5. Verify

Force one error in dev: appears ONCE in Sentry with requestId + readable
stack, and the same requestId query in Logs Insights finds the log story.
Force a queue failure: item lands in DLQ AND Sentry, counted once.
