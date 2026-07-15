---
name: amplitude
description: Amplitude SDK conventions — one wrapper, identify/reset lifecycle, user vs event properties, EU residency and ad-blocker proxying, no PII, dev/test modes.
---

# Amplitude

Library conventions; WHAT to track and the event-naming catalog is the
`add-analytics-event` skill — install both together.

## 1. One wrapper, one init

Features import [src/analytics/client.ts — adapt], never `@amplitude/*`
directly — swapping vendors or adding a second destination stays a one-file
change.

```ts
init(env.AMPLITUDE_KEY, {
  serverZone: 'EU',                    // [adapt — data residency decision]
  serverUrl: env.ANALYTICS_PROXY_URL,  // [adapt — proxy, see §4]
  defaultTracking: { pageViews: false, sessions: true }, // page views are
  // OUR explicit events (new-page), not URL-based auto-noise [house choice]
  minIdLength: 1,
})
```

Decisions made once, in code, not SDK defaults nobody chose: autocapture
on/off, session timeout, residency.

## 2. Identity lifecycle (ties to auth-patterns)

- Login/session restore: `setUserId(internalUserId)` — the internal ID,
  NEVER email
- **Logout: `reset()`** — this is on the auth-patterns logout checklist;
  missing it merges the next user's events into the previous identity
- Anonymous → login on the same device: the SDK merges device→user;
  don't "help" by re-sending history

## 3. User properties vs event properties

- **User properties** (`identify`): slow-changing facts — plan, role,
  locale, feature cohorts. Set on change, not on every event
- **Event properties**: facts about THIS action — snake_case, from the
  typed catalog (add-analytics-event)
- Never: PII (email, name, free text) in either; user properties that
  actually belong on the event (`current_page` as a user property is a bug)

## 4. Delivery reality

- Ad blockers eat direct Amplitude calls — measure-critical apps proxy
  through [first-party endpoint — adapt]; either way, analytics failures
  NEVER break UX (wrapper swallows its own errors, reports a metric,
  moves on)
- `flush()` on logout/unload for the events that matter at exit
- Cost/quota: event volume is billed — a `mousemove`-level event stream
  fails review; track outcomes (add-analytics-event's fire-on-outcome rule)

## 5. Dev & test modes

- Dev: log-events-to-console mode [adapt: wrapper no-ops the network and
  logs] — devs verify names/props without polluting the project
- Tests: mock the WRAPPER (one seam), assert calls — never mock Amplitude
  internals (testing-library-react's boundary rule)
- Verify in PR: new event visible once (not duplicated) in the Amplitude
  debugger from a staging session
