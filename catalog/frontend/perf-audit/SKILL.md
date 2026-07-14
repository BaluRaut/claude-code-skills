---
name: perf-audit
description: Performance pass on a page or app — measure first, then bundle size, re-renders, images, and lists. Report before/after numbers.
---

# Performance audit

Rule zero: **measure before touching anything.** Optimizations without a
baseline number are folklore. Report before/after for everything you change.

## 1. Baseline

- Lighthouse (prod build: `pnpm build && pnpm preview`) — record LCP, TBT, CLS
- React DevTools Profiler on the slow interaction — record what renders and why

## 2. Bundle (usually the biggest win)

- Analyze: [vite-bundle-visualizer / nx bundle analysis — adapt]
- Every route lazy-loaded? (see new-page)
- Heavy libs: charting/date/editor libs pulled into the main chunk → dynamic
  import; lodash → per-method imports; moment → replace [per repo policy]
- Duplicate dependencies (two versions of the same lib) → dedupe in lockfile

## 3. Re-renders

- Profiler: find components rendering on unrelated state changes
- Fix causes before memoizing: state too high in the tree, new object/array/
  function props created each render, context carrying too much
- `memo`/`useMemo`/`useCallback` ONLY on components the profiler showed as
  hot — blanket memoization is noise with a maintenance cost

## 4. Assets & lists

- Images: sized (`width`/`height` — CLS), lazy-loaded below the fold, modern
  format via [repo's image pipeline — adapt]
- Long lists (>~100 rows): virtualize [react-window / design-system table —
  adapt]; paginated queries where the API supports it

## 5. Report

Before/after table (LCP, TBT, bundle kB, render counts) in the PR. If a
change shows no measurable win, revert it — complexity without numbers loses.
