---
name: datadog-rum
description: Datadog RUM conventions — deliberate init/sampling, one error owner with Sentry, APM trace correlation via allowedTracingUrls, router-aware views, privacy defaults, cost control.
---

# Datadog RUM

Real User Monitoring: performance, sessions, and frontend→backend trace
correlation. Same wrapper rule as everything else: features import
[src/lib/monitoring.ts — adapt], never `@datadog/browser-rum` directly.

## 0. Decide the ERROR OWNER first (or pay twice and trust neither)

If the repo also runs Sentry (error-monitoring): ONE tool owns error
tracking/alerting; RUM owns performance/sessions/traces. Double-reporting
the same exceptions gives two half-accurate error rates and two bills.
[House split — adapt: Sentry = errors, Datadog = RUM + APM correlation is
the common one. If Datadog owns errors too, error-monitoring's rules apply
HERE and Sentry goes.]

## 1. Init — every value a decision

```ts
datadogRum.init({
  applicationId: env.DD_APP_ID,
  clientToken: env.DD_CLIENT_TOKEN,     // client tokens are public-safe by design
  site: 'datadoghq.eu',                 // [adapt — residency]
  service: 'acme-portal',
  env: env.STAGE,
  version: env.APP_VERSION,             // ties sessions to deploys — required
  sessionSampleRate: 100,               // [adapt — cost lever #1]
  sessionReplaySampleRate: 20,          // [adapt — cost lever #2; 100% replay is a bill]
  defaultPrivacyLevel: 'mask-user-input', // DoD minimum; 'mask' for stricter
  trackUserInteractions: true,          // action names need clean DOM/testids
  allowedTracingUrls: [env.API_BASE_URL], // §2 — the reason RUM earns its place
})
```

`setUser({ id: internalUserId })` on login, `clearUser()` on logout
(auth-patterns checklist) — internal ID only, no email/name.

## 2. The killer feature: frontend↔backend trace correlation

`allowedTracingUrls` makes RUM inject trace headers into http-client
requests — a slow page connects to the exact Lambda/APM trace behind it.
Requirements:

- The API's CORS must allow the injected headers
  (`x-datadog-trace-id`, `traceparent`) — coordinate with the backend repo
  or requests fail silently
- Backend services run the Datadog tracer with the same `env`/`service`
  tagging conventions
- This correlation replaces "works on my machine" debates with one trace —
  it's the main justification for RUM's cost

## 3. Views, actions, errors

- SPA views: track on route change wired to react-router [adapt: manual
  `startView` on route change with route PATTERN names (`/orders/:id`),
  not raw URLs — raw URLs shard every ID into its own view]
- Custom actions (`addAction`) follow the SAME naming as the analytics
  catalog (add-analytics-event) — product analytics stays in Amplitude;
  RUM actions are for performance/debugging context, not a second
  analytics system
- `addError` only from the wrapper, honoring §0's owner decision

## 4. Cost control (RUM decays into a big bill quietly)

- Sample rates reviewed when traffic grows — 100/100 defaults left
  unattended are the classic surprise invoice
- Replay only where debugging value is real [adapt: internal apps 100%,
  high-traffic public pages sampled low]
- Session quota alerting on in Datadog itself; one named owner glances
  monthly (same rotation as error-monitoring's alert hygiene)

## 5. Verify

Staging session: view names are route patterns, user id set after login
and cleared after logout, one API call shows a connected backend trace,
replay masks every input. Deploy: new `version` visible in RUM within one
release.
