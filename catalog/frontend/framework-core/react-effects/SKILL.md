---
name: react-effects
description: useEffect discipline and custom hook conventions — the decision tree before writing an effect, honest dependency arrays, mandatory cleanup, and the effect anti-patterns that cause most React bugs.
---

# React effects & custom hooks

Most React bugs live here. The rule: **an effect is a last resort** —
synchronizing with something OUTSIDE React (DOM APIs, subscriptions, timers,
analytics). Everything else has a better home.

## 1. The decision tree (run it BEFORE writing `useEffect`)

1. **Derived from props/state?** → compute in render (or `useMemo` if
   measured-expensive). Not an effect + setState.
2. **Response to a user action?** → event handler. "Effect watching a
   state flag the handler set" is a handler wearing a disguise.
3. **Server data?** → TanStack Query (see tanstack-query). Fetching in
   effects loses caching, dedupe, races, cancellation.
4. **Resetting state when a prop changes?** → `key={id}` remount, not an
   effect that mirrors props into state.
5. **Actually syncing with an external system?** → OK, `useEffect` — with
   the rules below.

## 2. Effect rules

- **Deps are honest.** `eslint react-hooks/exhaustive-deps` is never
  disabled; if the deps feel wrong, RESTRUCTURE (move the function inside
  the effect, `useCallback` upstream, or split the effect) — don't lie to
  the array
- **Cleanup is mandatory** for everything you start: listeners,
  subscriptions, intervals, observers — and in-flight work via
  `AbortController` so late responses don't setState after unmount
- One concern per effect — two unrelated syncs = two effects
- StrictMode double-invokes effects in dev **on purpose**: if double-run
  breaks it, cleanup is missing — fix the effect, never remove StrictMode
- Effect chains (effect sets state → triggers next effect → sets state) are
  a redesign signal: derive the values or move logic into the event

## 3. Custom hooks

- Extract when logic (not JSX) repeats or a component's effects crowd it:
  `useDebounce`, `usePageVisibility`, `useFeatureData`
- A hook owns its whole lifecycle — it starts AND cleans up; callers never
  clean up for it
- Return a stable API: memoize returned functions/objects if consumers will
  put them in deps arrays
- Location: [src/hooks/ shared; feature hooks next to the feature — adapt]
- Name says what it gives, not how (`useOrderPolling`, not
  `useIntervalFetchEffect`)

## 4. Review catches (the classics)

- `useEffect(() => { setX(compute(props)) }, [props])` — derived state,
  delete the effect
- Fetch in effect with no abort — race: fast navigation renders stale data
- `[]` deps with values used inside — stale closure bug hidden by the lint
  suppression above it
- `setInterval` without clear; listener without remove
- Object/array literal in deps recreated every render — infinite loop or
  useless effect churn; memoize or move it inside

## 5. Verify

Dev run WITH StrictMode: no double-fire bugs, no console warnings. Lint
clean with exhaustive-deps as error. For hooks: a test that unmounts
mid-flight and asserts no state-update warnings.
