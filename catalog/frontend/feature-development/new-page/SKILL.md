---
name: new-page
description: Add a new page/route — lazy-loaded route registration, auth guard, data loading, loading/error/empty states, document title, page-view analytics, i18n, and testids.
---

# New page

## 1. Scaffold

Create the page folder per repo convention: `src/pages/<feature>/` with
`<feature>-page.tsx` + spec. Pages compose — business logic lives in hooks,
UI in components.

## 2. Register the route (lazy)

In [src/router.tsx — adapt to repo]:

```tsx
const RefundsPage = lazy(() => import('@/pages/refunds/refunds-page'))
// inside routes:
{ path: '/refunds', element: <RefundsPage />, /* wrap with guard below */ }
```

- Lazy-load every page — never a static import in the router.
- Auth: wrap with the repo's guard ([`<RequireAuth>` / route loader — adapt]).
  Decide explicitly: public or authenticated? Which roles?

## 3. Data + the four states

Data comes from hooks in [the data layer — adapt]. Every page handles all
four states — missing ones are the #1 review comment:

1. **Loading** — skeleton from the design system, not a spinner-only page
2. **Error** — error component with retry, error reported to monitoring
3. **Empty** — designed empty state with a next action, not a blank table
4. **Success**

## 4. Page chrome

- Document title: [repo's title hook/helper] — translated, not hardcoded
- Page-view analytics event per the tracking catalog
- Breadcrumb/nav entry if the app has one

## 5. DoD pass

- All user-visible strings through i18n
- `data-testid` on interactive elements and assertion targets
- Unit test: renders each of the four states
- Playwright: add the flow or note "E2E deferred: <ticket>"

## 6. Verify

Run the repo verify command, then open the page via `pnpm dev` and check all
four states (throttle/block the API call to see loading/error).
