---
name: frontend-security
description: Frontend security rules — XSS and dangerouslySetInnerHTML policy, nothing secret in the bundle, token storage, dependency hygiene, link/PII safety.
---

# Frontend security

The frontend rules a reviewer (and the pre-push AI review) can enforce.
Backend enforcement is the real security boundary — everything here is
defense-in-depth plus not-leaking.

## 1. XSS

- React escapes rendered text by default — the bugs come from the escape
  hatches:
- `dangerouslySetInnerHTML` requires BOTH sanitization
  (`DOMPurify.sanitize(html)` [adapt]) and a comment justifying why HTML
  rendering is needed. Unsanitized = review blocker, no exceptions
- Never build HTML/CSS strings from user input; never `eval`/`new Function`
  on anything dynamic
- URLs from data: validate protocol before using as `href`/`src` —
  `javascript:` URLs are the classic hole
- External links with `target="_blank"`: `rel="noopener noreferrer"`

## 2. Nothing secret ships in the bundle

- Every `VITE_`/build-time var is PUBLIC — anyone can read it. "Secret" API
  keys for third parties belong behind a backend proxy endpoint, full stop
- Grep check in review: no tokens, no privileged keys, no internal URLs
  that shouldn't be public in the diff
- Frontend permission checks are UX, not security — hiding the button
  doesn't protect the endpoint; the backend authorizes (see auth-patterns)

## 3. Token & sensitive storage [house choice — adapt]

- Preferred: httpOnly secure cookies (JS can't read them — XSS can't steal
  them)
- If localStorage tokens are the existing architecture: acknowledge the
  XSS trade-off explicitly, keep expiry short, and logout clears it
  (auth-patterns) — don't ALSO cache user PII in localStorage
- Never in: URLs/query params (they hit logs, history, referrers)

## 4. Dependencies

- `pnpm audit` triage on the upgrade-deps cadence: fix criticals now,
  schedule the rest — don't let the output become wallpaper
- New dependency bar (ties to lodash-utils native-first): trivial utilities
  don't earn an install; check maintenance/provenance before adding, exact
  name (typosquatting is real)
- Lockfile changes in a PR must be explainable by the declared dep changes

## 5. PII & logging

- No PII in console logs, analytics properties (add-analytics-event), error
  reports (error-monitoring scrubs, but don't feed it either), or session
  replay (mask by default)
- Sensitive form fields: proper `autocomplete` attributes; no PII in
  data-testids or DOM attributes that don't need it

## 6. Verify

Diff pass: any `dangerouslySetInnerHTML`? any new env var that's actually a
secret? tokens/PII in storage, URLs, or logs? `pnpm audit` clean at the
agreed level. These four checks are in the pre-push review rubric.
