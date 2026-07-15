---
name: react-router-v6
description: React Router v6 conventions — central route config, lazy loading, guards, typed params, navigation rules, and the data-APIs decision.
---

# React Router v6

Version: [6.x — adapt; if the repo is on 6.4+ data routers vs plain
`<Routes>`, that changes the advice below — check how `router.tsx` is built].

## 1. One route config

All routes declared in [src/router.tsx — adapt] — no `<Route>` scattered in
feature components. Every page lazy-loaded (see new-page). Route paths as
constants/builders [src/routes.ts — adapt]:

```ts
export const routes = {
  orders: '/orders',
  orderDetail: (id: string) => `/orders/${id}`,
}
```

`navigate(routes.orderDetail(id))` — never template strings inline; renames
become one-line changes.

## 2. The data decision (make it once, repo-wide)

House position [adapt]: **TanStack Query owns data** (see tanstack-query);
loaders — if used at all — only call `queryClient.prefetchQuery` so
navigation warms the cache. Components stay the single reading path.
Mixing "some pages loader-fetch, some query-fetch" is the confusing state —
don't drift there PR by PR.

## 3. Navigation rules

- Rendered navigation = `<Link>`/`<NavLink>` (real anchors: cmd-click,
  middle-click work) — a `<div onClick={navigate}>` fails a11y AND UX review
- `useNavigate` only for imperative flows (after submit success, guards)
- Redirects preserve intent: guard sends `state={{ from: location }}`,
  login returns to it
- `useSearchParams` for filter/tab state that should survive refresh and be
  shareable — not useState (product decision per screen, but the default is
  URL for list filters)

## 4. Guards & layout routes

- Protected areas: one wrapper route (`<RequireAuth>` around an `<Outlet/>`
  layout route) — not a check inside every page
- Shared chrome (nav, breadcrumbs) = layout routes with `<Outlet/>`, not
  copy-paste per page
- Always a catch-all `path="*"` NotFound page — the router silently
  rendering nothing is a real production bug class

## 5. Params & pitfalls

- `useParams` returns `string | undefined` — parse/validate at the page
  boundary (zod-schemas), don't `!` it through the tree
- Relative links inside nested routes: know your base — `..` behavior
  changed in v6; test nested navigation explicitly
- `useEffect` on `location` to react to navigation is usually a smell —
  key the component or derive from params instead
