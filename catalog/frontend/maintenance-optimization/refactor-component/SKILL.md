---
name: refactor-component
description: Refactor a component safely — tests green before starting, behavior frozen during, extraction patterns, and a diff review for accidental changes.
---

# Refactor a component

Refactor = behavior identical, structure better. If behavior changes, that's
a feature/fix PR — don't mix the two.

## 1. Safety net BEFORE touching code

- Run the component's tests. Green? Proceed. No tests or thin tests? **Write
  behavior tests first** (see write-unit-tests) against the CURRENT behavior —
  they're your proof the refactor changed nothing
- Note the public surface: props API, emitted events, testids, analytics
  events — all frozen unless consumers are updated in the same PR

## 2. Extraction patterns (in order of preference)

1. **Logic → custom hook**: stateful logic tangled in JSX → `use<Feature>()`
   next to the component; component becomes mostly rendering
2. **JSX blocks → subcomponents**: a render helper function or a 300-line
   return → named child components with explicit props
3. **Conditional hell → early returns / state enum**: nested ternaries →
   `if (isLoading) return <Skeleton/>` chain, or a discriminated union state
4. **Prop drilling ≥3 levels → composition** (children/slots) before reaching
   for context

## 3. Keep it mechanical

Small steps, tests after each. If a step needs a behavior decision ("was this
edge case intended?") — stop and ask; that's a bug discovery, not a refactor
choice. Note it, preserve current behavior, file it.

## 4. The paranoid diff review

Before pushing, read the full diff hunting for ACCIDENTAL changes: dropped
props, lost `key`s, changed conditions (`>=` → `>`), lost i18n keys, dropped
testids/analytics. These are the refactor killers.

## 5. Verify

Tests green (unchanged test files = strongest signal), repo verify command,
and exercise the component in the running app — all its states.
