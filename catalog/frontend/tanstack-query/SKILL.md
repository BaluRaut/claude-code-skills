---
name: tanstack-query
description: TanStack Query (React Query) conventions — defaults, key factories, dependent/derived queries, invalidation, optimistic updates, and v5 API notes.
---

# TanStack Query

Version: [v5.x — adapt; v4→v5 renamed `isLoading`→`isPending`, `cacheTime`→
`gcTime`, and made the object signature mandatory]. Task-level procedure for
adding a hook lives in the `new-data-hook` skill — this skill is the library
conventions behind it.

## 1. One QueryClient, deliberate defaults

Defaults live in one place [src/lib/query-client.ts — adapt], not per-hook:

```ts
new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,        // decide consciously; 0 means refetch storms
      retry: 1,                  // and never retry 4xx — check error.status
      refetchOnWindowFocus: false, // [house choice — adapt]
    },
  },
})
```

## 2. Patterns

- **Keys**: always from the key factory (see new-data-hook) — never inline arrays.
- **Dependent queries**: `enabled: !!userId` — never conditionally call hooks.
- **Derived data**: `select` on the query, not `useMemo` in every consumer:
  `useQuery({ ...opts, select: (data) => data.items.length })` — components
  re-render only when the selected slice changes.
- **Mutations**: every mutation declares invalidation in `onSuccess`.
  Optimistic updates only where UX demands it, and always with rollback:
  `onMutate` (snapshot + setQueryData) / `onError` (restore) / `onSettled`
  (invalidate).
- **Prefetch**: route-level or on-hover via `queryClient.prefetchQuery` for
  the screens users hit next — cheap wins.

## 3. Boundaries

- Server state ONLY. UI state (modals, filters-in-progress) stays in
  useState/[zustand — if repo has it]. If it came from an API, it lives here;
  if the user typed it and hasn't saved, it doesn't.
- Errors propagate to the error boundary via `throwOnError` [house choice —
  adapt] or render local error states (see new-page's four states) —
  pick ONE convention per repo.

## 4. Pitfalls (review catches)

- `useEffect` + `refetch()` to "sync" — almost always wrong; fix the key so
  the query re-runs when inputs change
- Copying query data into useState — you now have two sources of truth
- Infinite refetch loops from unstable key parts (inline objects — memoize
  or serialize filters)

## 5. Verify

React Query Devtools open: no duplicate keys for the same data, no queries
stuck refetching, cache updates after mutations without a manual reload.
