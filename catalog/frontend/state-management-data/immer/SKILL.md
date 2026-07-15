---
name: immer
description: Immer and useImmer conventions — when produce earns its place, draft rules, useImmer/useImmerReducer for local state, zustand/RTK integration, classic pitfalls.
---

# Immer / useImmer

Version: [immer v10.x + use-immer — adapt]. Immer is for **complex nested
updates**. Flat state updated with a spread doesn't need it — `produce` on a
two-field object is noise. The signal it earns its place: spreads nested
three levels deep (`{...s, a: {...s.a, b: {...s.a.b, x}}}`).

## 1. Draft rules (where the bugs live)

```ts
const next = produce(state, (draft) => {
  const item = draft.items.find((i) => i.id === id)
  if (item) item.status = 'approved'     // mutate the draft freely
})
```

- **Mutate OR return — never both.** And the arrow-function trap:
  `produce(s => s.count = 1)` implicitly RETURNS the assignment → immer
  throws. Always use braces: `produce(s => { s.count = 1 })`.
- Never let the draft escape `produce` (stored in a variable used later,
  passed to a callback, awaited across) — drafts are revoked after the
  producer runs; touching one later throws.
- No `async` producers; compute async results first, then produce.
- `Map`/`Set` in state need `enableMapSet()` once at startup [adapt];
  class instances need `[immerable] = true` — prefer plain objects.

## 2. React local state: useImmer / useImmerReducer

```tsx
import { useImmer } from 'use-immer'

const [form, updateForm] = useImmer({ items: [], filters: { status: 'all' } })
updateForm((draft) => { draft.filters.status = 'open' })
```

- `useImmer` replaces useState when updates are nested — same rules as above
- `useImmerReducer` for multi-action local state: reducers mutate the draft,
  no spread pyramids
- Structural sharing is the hidden win: untouched branches keep reference
  identity, so memoized children skip re-renders — don't "fix" this by
  cloning

## 3. Where immer is ALREADY there (don't double-wrap)

- **Redux Toolkit**: `createSlice` reducers run inside immer — mutate
  directly; wrapping in `produce` again or returning spreads is wrong on
  both sides (see redux-toolkit)
- **zustand**: via middleware — `create(immer((set) => ...))`, then
  `set((draft) => { draft.x = 1 })` [if the repo adopted it — adapt; without
  the middleware, zustand `set` is shallow-merge, NOT a draft]

One style per repo: if RTK/zustand-immer is the state layer, standalone
`produce` calls should be rare and justified.

## 4. Pitfalls (review catches)

- `produce` for a one-level update — plain spread reads better
- Mutating the ORIGINAL state object anywhere (immer protects the draft,
  not your discipline elsewhere — `state.items.push()` outside produce is
  still the bug it always was)
- Logging drafts (`console.log(draft)`) — you get Proxy noise; use
  `current(draft)` for debugging snapshots
- Spreading a draft (`{...draft.item}`) mid-producer then mutating the
  spread — the mutation silently doesn't land in the result

## 5. Verify

Unit tests assert both the change AND non-mutation of the input
(`expect(next).not.toBe(prev)` + untouched branch `toBe` equality). Type
errors about `WritableDraft` usually mean a draft leaked — fix the leak,
don't cast.
