---
name: debug-prod-error
description: Production triage runbook — stabilize first, find the story via request ID, isolate the layer, fix root cause with a regression test, close the loop.
---

# Debug a production error

Task skill — the runbook. Rule zero (mirrors debug-ui-bug): **no fix
without understanding.** Rule one: **stabilize before root-causing** —
if a deploy correlates, roll back FIRST (deploy skill), debug after.

## 1. Scope it (5 minutes, before any code)

- Since when? All users or one tenant? One endpoint or everything?
- Correlate with: last deploys (any service in the path), alarms that
  fired, AWS health [adapt: status dashboard]
- Widespread + deploy-correlated → rollback now. Narrow → continue

## 2. Get the story — request ID first

From the report/frontend error (error-monitoring carries the request id):

```
# CloudWatch Logs Insights, the bread-and-butter query [adapt log group names]
fields @timestamp, level, msg, requestId, err.message
| filter requestId = 'req_abc123'
| sort @timestamp asc
```

No request id? Query the error class instead: filter level='error' over
the window, group by err.message — find the top offender and pull ONE
full requestId story from it. Traces [adapt: X-Ray/Datadog] for
cross-service timing; DLQ messages are pre-packaged repro cases (sqs §4).

## 3. Isolate the layer (same outside-in as debug-ui-bug)

1. **Ingress**: did the request arrive malformed? (validation 400s spiking
   = client/contract bug, different fix)
2. **Auth/config**: expired secret, rotation window (secrets-config §3),
   missing env after deploy
3. **Code**: the exception path — reproduce locally with the event JSON
   from logs/DLQ (node-testing fixtures)
4. **Downstream**: DB throttles/connections (rds-postgres §1, dynamodb hot
   keys), third-party outage, timeout chain (lambda-patterns §3)

## 4. Fix = root cause + regression test + guardrail

- The one-sentence cause goes in the PR (debug-ui-bug's rule)
- The event/request that broke it becomes a test fixture in the same PR
- Ask: which ALARM should have caught this before the user did? Add it
  (logging-observability §5) — that's the difference between an incident
  and a recurring incident
- Then: redrive the DLQ / backfill what the bug dropped — data cleanup is
  part of the fix, not an afterthought

## 5. Close the loop

Post the summary (what broke, why, user impact, guardrail added) in
[the channel — adapt]. Two of the same class of incident = a skill or
CLAUDE.md rule update (common-rules: wrong twice → into the file).
