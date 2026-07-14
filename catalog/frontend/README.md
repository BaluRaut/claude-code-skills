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
| theming | tokens as single source → styled-components + antd, dark mode |
| styled-components | module-level defs, transient props, typed theme, pitfalls |
| antd | version-pinned; ConfigProvider tokens, never .ant-* overrides |

`new-component` isn't duplicated here — the two variants (nx / plain Vite)
already live in `../samples/frontend-react-nx/` and `../samples/frontend-react/`.
