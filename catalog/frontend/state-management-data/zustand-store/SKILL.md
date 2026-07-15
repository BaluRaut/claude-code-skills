---
name: zustand-store
description: Zustand conventions — client-state only, feature stores, selector discipline, actions co-located, persist/devtools middleware, testing.
---

# Zustand

Version: [v4/v5 — adapt]. Zustand holds **client state that outlives a
component** (wizard progress, UI preferences, cross-cutting selections).
Server data belongs in TanStack Query — a fetch result inside a zustand
store is a bug (see tanstack-query skill).

## 1. Store shape — per feature, not one god-store

```ts
// [src/stores/checkout-store.ts — adapt]
interface CheckoutState {
  step: number
  selectedItems: string[]
  // actions live IN the store — components never setState raw
  nextStep: () => void
  toggleItem: (id: string) => void
  reset: () => void
}

export const useCheckoutStore = create<CheckoutState>()((set) => ({
  step: 0,
  selectedItems: [],
  nextStep: () => set((s) => ({ step: s.step + 1 })),
  toggleItem: (id) => set((s) => ({ selectedItems: toggle(s.selectedItems, id) })),
  reset: () => set({ step: 0, selectedItems: [] }),
}))
```

- One store per feature domain; delete the store when the feature dies
- Every store has a `reset()` — and it's called on logout [adapt: central
  reset registry]

## 2. Selector discipline (the perf part)

- Components subscribe to slices, never the whole store:
  `useCheckoutStore((s) => s.step)` — `useCheckoutStore()` bare re-renders
  on every change
- Multiple values: `useShallow` (v5) / shallow equality — not an inline
  object selector, which re-renders always

## 3. Middleware

- `devtools()` in development [adapt policy]; name the store for the panel
- `persist()` only for genuinely persistent prefs — with `version` +
  `migrate` from day one, and NEVER persist server data or anything sensitive
- Order: `devtools(persist(...))` — outermost wraps

## 4. Testing

Stores are plain hooks: test actions via `useCheckoutStore.getState()` +
assertions on state transitions; `store.setState` for arrange. Reset in
`beforeEach` (persist middleware leaks between tests otherwise).

## 5. Pitfalls

- Mirroring props/query data into the store "for convenience" — two sources
  of truth
- Business logic in components calling multiple `set`s — make it one named
  action
- A second store importing another store's state inside `set` — extract
  shared logic instead
