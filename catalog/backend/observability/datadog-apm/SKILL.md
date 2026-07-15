---
name: datadog-apm
description: Datadog on Lambda — extension layer via the serverless plugin, unified service tagging, trace correlation with RUM, log injection into pino, custom metrics, cost control.
---

# Datadog APM (Lambda)

Backend half of the trace story — datadog-rum's `allowedTracingUrls`
(frontend catalog) only pays off when these functions run the tracer with
matching tags.

## 0. One error owner

Same decision as sentry-node §0 — if Sentry owns exceptions, Datadog owns
APM/metrics/traces here. Write the split in CLAUDE.md.

## 1. Wiring — the plugin, not hand-rolled layers

[adapt: serverless-plugin-datadog] in serverless.yml:

```yaml
custom:
  datadog:
    site: datadoghq.eu            # [adapt — must match RUM's site]
    apiKeySecretArn: ${ssm:/shared/datadog/api-key-arn}   # never plaintext
    env: ${sls:stage}
    service: ${self:service}
    version: ${env:GIT_SHA}
    enableDDTracing: true
    captureLambdaPayload: false   # payloads carry PII — off unless scrubbed [adapt]
```

- The Extension layer ships traces/metrics directly — no CloudWatch
  log-forwarder latency/cost [adapt if the account standardized otherwise]
- **Unified tagging (`env`/`service`/`version`) is the whole game** —
  RUM sessions, APM traces, and logs join on these three; a service with
  ad-hoc tags is invisible to the correlation story
- Don't double-instrument: X-Ray off when dd-trace is on [adapt]

## 2. Log injection — traces ↔ pino

Enable DD trace-log injection [adapt: `DD_LOGS_INJECTION` /
dd-trace logger integration] so `trace_id`/`span_id` land in every pino
line — from a slow RUM session: trace → spans → the exact log lines. This
plus the requestId (logging-observability §2) makes frontend-to-DB
debugging one click-through.

## 3. Custom metrics & spans

- Business/failure-mode metrics via the extension
  [adapt: `datadog-lambda-js` `sendDistributionMetric` or logs-to-metrics]
  — the metric set comes from logging-observability §5, this is just the
  transport
- Custom spans (`tracer.trace('orders.calculate', ...)`) only around
  measured-hot sections — span-everything is noise with overhead

## 4. Cost control (same decay curve as RUM)

APM is billed per invocation/host + indexed spans: sampling rules for
high-volume low-value endpoints (health checks EXCLUDED from tracing
[adapt]), log indexing filters, and the same monthly named-owner review
as datadog-rum §4 and error alert hygiene.

## 5. Verify

One staging request: RUM session → backend trace → pino lines with
trace_id, all joined, tags env/service/version consistent. Alarm/monitor
on 5xx + p99 exists per function class (logging-observability §5).
Datadog bill line item reviewed once after enabling — not discovered.
