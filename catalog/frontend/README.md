# Frontend skill catalog

**How to actually use these → [USAGE.md](USAGE.md)** — real cases, three repo
conditions (legacy / modern monorepo / greenfield), and the 4-week adoption loop.

A menu of 30 skills covering the common frontend tasks and libraries,
organized by category. **Nothing here goes into a repo verbatim** — this
folder costs nothing sitting here; installed skills cost context in every
session. Process:

1. Pick candidates devs think they'd use (start with 5–8, not all).
2. Adapt placeholders (`[...]` markers) to the repo's REAL code — libraries,
   paths, commands, versions. A skill that contradicts the repo makes Claude worse.
3. Copy the skill folder into the repo's `.claude/skills/<name>/` (or the
   shared plugin repo if it applies everywhere).
4. After a month: keep what got used, delete what didn't, promote winners to
   the shared plugin.

Two kinds of skills, complementary:
- **Task skills** — a procedure end-to-end ("add a page", "fix a flaky test")
- **Library skills** — the house conventions for one library ("how we use
  TanStack Query"). Install only for libraries the repo ACTUALLY uses, and
  where alternatives exist (vitest/jest, zustand/redux) — exactly one.

## Feature Development — [feature-development/](feature-development/)

| Skill | Covers |
|---|---|
| new-page | route, lazy load, guard, the four states, page-view event |
| new-form | schema validation, submit states, server errors, a11y |
| add-feature-flag | gate code, default off, cleanup ticket |
| add-translation | i18n keys, plurals/interpolation, locale TODOs |
| add-analytics-event | typed catalog, naming, when to fire, test |

## State Management & Data Fetching — [state-management-data/](state-management-data/)

| Skill | Covers |
|---|---|
| new-data-hook | task: add a React Query hook (keys, invalidation, mock, test) |
| tanstack-query | library: defaults, key factories, optimistic updates |
| zustand-store | client-state only, feature stores, selector discipline |
| redux-toolkit | slices, typed hooks, what does NOT go in the store |
| immer | produce/useImmer draft rules, RTK/zustand integration, pitfalls |

## Styling & Design Systems — [styling-design-systems/](styling-design-systems/)

| Skill | Covers |
|---|---|
| theming | tokens as single source → styled-components + antd, dark mode |
| styled-components | module-level defs, transient props, typed theme |
| antd | version-pinned; ConfigProvider tokens, never .ant-* overrides |

## Validation & Forms — [validation-forms/](validation-forms/)

| Skill | Covers |
|---|---|
| zod-schemas | schema = the type, parse at boundaries, i18n messages |

## Testing & Quality Assurance — [testing-qa/](testing-qa/)

| Skill | Covers |
|---|---|
| write-unit-tests | task: tests derived from acceptance criteria |
| write-e2e-test | task: a Playwright spec for a user flow |
| fix-flaky-test | classify the cause, fix it, prove it |
| add-testids | data-testid sweep for Playwright |
| playwright-e2e | library: config, fixtures, selectors, trace debugging |
| vitest-unit | Vite-shared config, vi.mock hoisting, fake timers |
| jest-unit | TS/alias config, mock patterns (Jest repos only) |
| testing-library-react | query priority, user-event, shared custom render |

## Framework Core — [framework-core/](framework-core/)

| Skill | Covers |
|---|---|
| react-router-v6 | central route config, guards, the data-APIs decision |

## Utilities & Performance — [utilities-performance/](utilities-performance/)

| Skill | Covers |
|---|---|
| lodash-utils | native-first policy, tree-shakeable imports, debounce traps |
| date-fns-utils | central helpers, parseISO, locales, dayjs coexistence |

## Maintenance & Optimization — [maintenance-optimization/](maintenance-optimization/)

| Skill | Covers |
|---|---|
| debug-ui-bug | reproduce → isolate → fix → regression test |
| perf-audit | measure-first bundle + re-render pass |
| a11y-audit | axe + keyboard + focus + labels pass |
| upgrade-deps | one major at a time, changelog-driven |
| refactor-component | tests green before, behavior frozen during |

---

Overlap by design: `write-e2e-test` is the PROCEDURE, `playwright-e2e` the
LIBRARY rules — a repo doing E2E installs both. Same for `write-unit-tests`
+ (`vitest-unit` or `jest-unit`) + `testing-library-react`, and
`new-data-hook` + `tanstack-query`.

`new-component` isn't duplicated here — the two variants (nx / plain Vite)
live in `../../samples/frontend-react-nx/` and `../../samples/frontend-react/`.
