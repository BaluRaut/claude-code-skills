---
name: lodash-utils
description: Lodash discipline — native-first policy, the short allowlist where lodash earns its place, tree-shakeable imports, React-specific debounce/throttle rules.
---

# Lodash

Policy skill: lodash is allowed, but native-first. Modern JS covers most of
classic lodash — every lodash call a reviewer sees should be one natives
can't do cleanly.

## 1. Native first (reviewer's translation table)

| Instead of | Use |
|---|---|
| `_.map/filter/find/some/every` | array methods |
| `_.uniq(arr)` | `[...new Set(arr)]` |
| `_.groupBy` | `Object.groupBy` [if runtime target allows — adapt] |
| `_.pick/omit` (small, static) | destructuring |
| `_.isNil(x)` / `_.get(a,'b.c')` | `x == null` / `a?.b?.c ?? fallback` |
| `_.cloneDeep` | `structuredClone` |

## 2. Where lodash EARNS its import [house allowlist — adapt]

- `debounce` / `throttle` — with leading/trailing/cancel semantics done right
- `groupBy` / `keyBy` when targets don't have `Object.groupBy`
- `isEqual` — real deep equality (but see §4)
- `merge` for deep config merging (sparingly — explicit spreads read better)

## 3. Imports — bundle rule (non-negotiable)

```ts
import debounce from 'lodash/debounce'   // or lodash-es named imports [adapt]
```

`import _ from 'lodash'` and `import { debounce } from 'lodash'` (CJS) ship
the whole library — perf-audit will find it; don't let it in.

## 4. React-specific traps

- `debounce` created in render is recreated every render — useless. Stabilize:
  `useMemo(() => debounce(fn, 300), [])` and `.cancel()` in the effect
  cleanup, or the debounced call fires after unmount (state-update warnings)
- `isEqual` inside `React.memo` comparators or effect deps: measure first —
  deep-comparing large props every render can cost more than re-rendering;
  usually the fix is stabilizing the upstream reference
- Debouncing user input? The i18n/analytics side matters too: fire analytics
  on the settled value, not every keystroke (add-analytics-event)
