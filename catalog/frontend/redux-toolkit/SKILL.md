---
name: redux-toolkit
description: Redux Toolkit conventions for repos that use Redux — slices, typed hooks, createSelector, what does NOT belong in the store, RTK Query boundary.
---

# Redux Toolkit

For repos with an existing Redux investment [version — adapt]. Do NOT
introduce Redux into a repo that doesn't have it — new repos here use
TanStack Query + zustand (see those skills). One repo, one client-state
library: RTK and zustand together is a review-blocker.

## 1. Non-negotiables (RTK era)

- `createSlice` only — no hand-written action types, action creators, or
  switch-statement reducers anywhere in new code
- Typed hooks from [src/store/hooks.ts — adapt]: `useAppSelector` /
  `useAppDispatch` — never raw `useSelector`/`useDispatch`
- State is normalized for collections: `createEntityAdapter`, not arrays
  you `.find()` through

## 2. What does NOT go in the store

- **Server data** — that's TanStack Query's job [or RTK Query if the repo
  standardized on it — adapt; never both fetching layers in one repo]
- Form-in-progress state (react-hook-form owns it), one-component UI state
  (useState), derived data (selectors compute it)

The store should shrink over time in repos migrating to query hooks — new
server-data slices are regressions.

## 3. Selectors

- Selectors exported next to the slice; components never reach into
  `state.foo.bar.baz` inline
- Derived/expensive selectors: `createSelector` — and keep input selectors
  stable; a `createSelector` that returns a new array every call memoizes
  nothing

## 4. Async

- `createAsyncThunk` with the three-state pattern handled in
  `extraReducers` (pending/fulfilled/rejected — all three, every time)
- Side effects beyond fetching: [listener middleware / sagas if legacy —
  adapt to what the repo already uses; don't add a second effects system]

## 5. Testing

Reducers are pure — test them directly: `reducer(prevState, action)` equals
expected state. Thunks: mock at the API boundary, assert dispatched actions.
Component tests get a real store from a `makeTestStore()` helper with
preloaded state [test-utils — adapt].
