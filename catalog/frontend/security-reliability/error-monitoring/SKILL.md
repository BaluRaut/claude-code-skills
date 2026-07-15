---
name: error-monitoring
description: Error tracking the house way — one monitoring wrapper, error boundary placement, no swallowed catches, context/severity rules, PII scrubbing, source maps, alert hygiene.
---

# Error monitoring

Tool: [Sentry — adapt]. One wrapper module [src/lib/monitoring.ts — adapt];
feature code imports the wrapper, never the vendor SDK — swapping/upgrading
the tool is then one file.

## 1. Init (once, at startup)

- `environment` + `release` set — errors without a release can't map to a
  deploy or source maps
- Sample rates decided consciously [adapt: errors 100%, performance/replay
  lower] — not the SDK defaults nobody chose
- `beforeSend` scrubs PII: no emails, names, free-text user input in
  messages/breadcrumbs; user context is the internal user ID only
- Session replay (if used): mask inputs by default (see the DoD)

## 2. Error boundaries

- Root boundary (last resort, full-page fallback) + one per route/page
  (see new-page) so a crashing widget kills a section, not the app
- Fallback UI from the design system with a retry/reload action — never a
  blank screen
- Boundaries report to monitoring with the component stack; the shared
  `<AppErrorBoundary>` [adapt] does this — no bespoke boundaries per feature

## 3. The catch rules (what reviewers enforce)

- **No swallowed errors**: every `catch` either meaningfully recovers
  (fallback value the UX actually supports) or reports + surfaces. Bare
  `catch (e) {}` and `catch (e) { console.log(e) }` fail review
- Severity: unexpected exceptions = `error`; expected-but-notable
  (retry exhausted, degraded fallback used) = `warning`; 4xx validation
  responses are NOT errors — they're user flow, don't pollute the feed
- Context on every report: feature tag, route, correlation/request ID from
  http-client so a frontend error joins its backend trace
- ApiError from http-client: report 5xx/network once, at the layer that
  handles it — not in the client AND the hook AND the boundary (triple
  counting makes rates meaningless)

## 4. Source maps & releases

Upload source maps in CI per release [adapt: Sentry CLI / vite plugin] —
minified stack traces are write-only noise. Verify after the next deploy:
one real stack trace must show original file/line before calling it done.

## 5. Alert hygiene (the part that decays)

- A NEW error type in prod → a ticket, same week — the feed stays
  actionable or it becomes wallpaper
- Noisy known errors: fix, or explicitly ignore with a code comment linking
  the decision — never silently mute in the tool's UI
- Weekly glance at top errors is a named person's job [adapt — rotate]
