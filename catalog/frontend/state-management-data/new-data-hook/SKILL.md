---
name: new-data-hook
description: Add a React Query hook for a new API interaction — query keys, hook shape, error normalization, cache invalidation, test mock.
---

# New data hook

All server state goes through React Query hooks in [src/api / libs/data-access
— adapt]. Components never call fetch/axios directly.

## 1. Query key — in the central keys file

```ts
// [keys.ts — adapt path]
export const refundKeys = {
  all: ['refunds'] as const,
  list: (filters: RefundFilters) => [...refundKeys.all, 'list', filters] as const,
  detail: (id: string) => [...refundKeys.all, 'detail', id] as const,
}
```

Never inline key arrays in components — invalidation breaks silently when
keys drift.

## 2. The hook

```ts
export function useRefunds(filters: RefundFilters) {
  return useQuery({
    queryKey: refundKeys.list(filters),
    queryFn: () => client.get<RefundsResponse>('/refunds', { params: filters }),
  })
}

export function useCreateRefund() {
  const qc = useQueryClient()
  return useMutation({
    mutationFn: (input: CreateRefundInput) => client.post('/refunds', input),
    onSuccess: () => qc.invalidateQueries({ queryKey: refundKeys.all }),
  })
}
```

Rules:
- Response/input types come from [shared API types — adapt]; no `any`.
- Mutations declare their invalidation — every mutation answers "what cached
  data does this stale?" in `onSuccess`.
- Errors: let them propagate to the repo's error handling; don't
  catch-and-console inside hooks.

## 3. Mock for tests

Add the [MSW handler / mock — adapt] next to the hook so component tests and
Storybook work without a backend.

## 4. Test

Hook test with a wrapper QueryClient: success shape, error state, and for
mutations — that the right keys are invalidated.

## 5. Verify

Repo verify command + exercise the consuming screen in dev.
