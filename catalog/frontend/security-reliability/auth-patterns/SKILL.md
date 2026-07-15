---
name: auth-patterns
description: Auth the house way — three-state session model, one useAuth source of truth, route guards and permission-based UI gating, single-flight refresh, and logout that clears everything.
---

# Auth patterns

Session model: [adapt — httpOnly cookie session / JWT + refresh]. One auth
module [src/auth/ — adapt] owns it all; auth state read anywhere else is a
review catch.

## 1. Three states, never a boolean

```ts
type AuthStatus = 'loading' | 'authenticated' | 'anonymous'
const { status, user, can } = useAuth()
```

`isLoggedIn: boolean` can't represent "still checking" — which is how apps
flash protected content or bounce users to login on every refresh. Guards
and pages branch on all three states (loading renders the app skeleton,
not a redirect).

## 2. Route guards (with react-router-v6)

- One `<RequireAuth>` layout route wrapping the protected tree — not a
  check inside every page; role-gated areas get `<RequireRole role="...">`
  variants
- Redirect to login carries `state={{ from: location }}`; login returns
  the user to where they were headed

## 3. UI gating — permissions, not role strings

```tsx
{can('refund:create') && <Button data-testid="refund-create">…</Button>}
```

- A `can(permission)` helper backed by the role/permission map [adapt] —
  scattered `user.role === 'admin'` checks are the review catch; roles
  change, call sites shouldn't
- Say it out loud in the code review: **UI gating is UX only** — the
  backend authorizes every request regardless (frontend-security)

## 4. Token lifecycle

- Refresh is single-flight and lives in http-client (see it) — the auth
  module exposes `ensureFresh`/`onSessionExpired`, nothing else duplicates
  refresh logic
- Session expiry → ONE behavior app-wide [adapt: redirect with return-to +
  toast], triggered centrally — not per-feature 401 handling

## 5. Logout clears EVERYTHING (the checklist)

One central `logout()` — the classic bug is the next user on a shared
machine seeing the previous user's cached data:

1. Server-side session/token revocation call
2. `queryClient.clear()` — TanStack Query cache
3. Every zustand store's `reset()` (zustand-store's registry)
4. Persisted storage: auth keys + any persisted store state
5. Monitoring/analytics user context cleared (error-monitoring,
   add-analytics-event)
6. Redirect to public route

## 6. Testing

- `renderWithProviders` accepts an auth state (testing-library-react):
  every guarded component gets a loading + anonymous + authorized test
- Playwright: `storageState` per role from global setup (playwright-e2e);
  one spec covers the actual login flow, one covers expiry → re-login
  return-to
