# Frontend skill catalog

**How to actually use these → [USAGE.md](USAGE.md)** — real cases, three repo
conditions (legacy / modern monorepo / greenfield), and the 4-week adoption loop.

A menu of skills covering the common frontend tasks. **Nothing here goes into
a repo verbatim** — this folder costs nothing sitting here; installed skills
cost context in every session. Process:

1. Pick candidates devs think they'd use (start with 4–5, not all).
2. Adapt placeholders (`[...]` markers) to the repo's REAL code — libraries,
   paths, commands. A skill that contradicts the repo makes Claude worse.
3. Install into the pilot repo's `.claude/skills/` (or the shared plugin repo
   if it applies everywhere).
4. After a month: keep what got used, delete what didn't, promote winners to
   the shared plugin. Cap per repo: ~5–8 installed skills.

## The menu

Two kinds of skills, complementary:
- **Task skills** — a procedure end-to-end ("add a page", "fix a flaky test")
- **Library skills** — the house conventions for one library ("how we use
  TanStack Query"). Install only the ones matching libraries the repo
  ACTUALLY uses — a redux-toolkit skill in a zustand repo actively harms.

### Task skills

| Skill | Covers |
|---|---|
| new-page | route, lazy load, guard, states, title, page-view event |
| new-data-hook | React Query hook, keys, invalidation, mock, test |
| new-form | schema validation, submit states, server errors, a11y |
| add-feature-flag | gate code, default off, cleanup ticket |
| add-analytics-event | typed catalog, naming, when to fire, test |
| add-translation | i18n keys, plurals/interpolation, locale TODOs |
| add-testids | data-testid sweep for Playwright |
| write-unit-tests | tests derived from acceptance criteria |
| write-e2e-test | Playwright spec for a user flow |
| fix-flaky-test | classify the cause, fix it, prove it |
| debug-ui-bug | reproduce → isolate → fix → regression test |
| perf-audit | measure-first bundle + re-render pass |
| a11y-audit | axe + keyboard + focus + labels pass |
| upgrade-deps | one major at a time, changelog-driven |
| refactor-component | tests green before, behavior frozen during |

### Library skills

| Category | Skill | Covers |
|---|---|---|
| State & data | tanstack-query | defaults, key factories, invalidation, optimistic updates |
| State & data | zustand-store | client-state only, feature stores, selector discipline |
| State & data | redux-toolkit | slices, typed hooks, what does NOT go in the store |
| Styling & design | theming | tokens as single source → styled-components + antd, dark mode |
| Styling & design | styled-components | module-level defs, transient props, typed theme |
| Styling & design | antd | version-pinned; ConfigProvider tokens, never .ant-* overrides |
| Validation & forms | zod-schemas | schema = the type, parse at boundaries, i18n messages |
| Testing & QA | playwright-e2e | config, fixtures, selector strategy, trace debugging |
| Testing & QA | vitest-unit | Vite-shared config, vi.mock hoisting, fake timers |
| Testing & QA | jest-unit | TS/alias config, mock patterns (Jest repos only) |
| Testing & QA | testing-library-react | query priority, user-event, shared custom render |
| Framework core | react-router-v6 | central route config, guards, the data-APIs decision |
| Utilities & perf | lodash-utils | native-first policy, tree-shakeable imports, debounce traps |
| Utilities & perf | date-fns-utils | central helpers, parseISO, locales, dayjs coexistence |

Overlap by design: `write-e2e-test` is the PROCEDURE, `playwright-e2e` the
LIBRARY rules — a repo doing E2E installs both. Same for `write-unit-tests`
+ (`vitest-unit` or `jest-unit`) + `testing-library-react`, and
`new-data-hook` + `tanstack-query`. Pick ONE per slot where alternatives
exist: vitest OR jest, zustand OR redux-toolkit — never both in one repo.

`new-component` isn't duplicated here — the two variants (nx / plain Vite)
already live in `../samples/frontend-react-nx/` and `../samples/frontend-react/`.
