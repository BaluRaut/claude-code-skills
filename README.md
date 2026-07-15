# Claude Code Skills — templates, catalog & rollout playbook

A practical kit for running an AI-enabled dev team with [Claude Code](https://claude.com/claude-code):
CLAUDE.md templates, a frontend skill catalog, and the process rules that make
"AI writes, developer reviews" actually work.

Built for a team of 12 devs shipping React apps (pnpm/nx) and Node serverless
services (Hono + Serverless Framework), on individual Claude Max plans — but
everything here is adaptable.

## Start here

- **[Team rollout plan](claude-code-team-plan.html)** — the 4-week plan:
  billing, per-repo foundation, shared skills, PR review, hooks, team habits
- **[Common rules](samples/common-rules.md)** — the stack-independent baseline
  every repo must satisfy (new or existing)
- **[Definition of Done](samples/definition-of-done.md)** — the per-change
  contract (i18n, analytics, error tracking, testids, tests) that replaces
  human nagging

## Repo templates

Copy into a repo's root, adapt the placeholders, commit:

- **[Backend — Hono + Serverless Framework v3](samples/backend-hono-serverless/CLAUDE.md)**
  — CLAUDE.md, settings.json with deny-list, `new-endpoint` + `deploy` skills
- **[Frontend — React in an nx monorepo](samples/frontend-react-nx/CLAUDE.md)**
  — CLAUDE.md, settings.json, `new-component` skill
- **[Frontend — standalone React (Vite)](samples/frontend-react/CLAUDE.md)**
  — CLAUDE.md, settings.json, `new-component` + `deploy` skills

See the [samples guide](samples/README.md) for how to apply them
(existing repo = derive from real code; new repo = impose the template).

## Frontend skill catalog (28 skills)

A menu — install 5–8 per repo, adapt placeholders first, prune monthly.

- **[Catalog & menu](catalog/frontend/README.md)**
- **[Usage playbook](catalog/frontend/USAGE.md)** — real cases, three repo
  conditions (legacy / modern monorepo / greenfield), the 4-week adoption loop

Building: [new-page](catalog/frontend/new-page/SKILL.md) ·
[new-data-hook](catalog/frontend/new-data-hook/SKILL.md) ·
[new-form](catalog/frontend/new-form/SKILL.md)
&nbsp;|&nbsp; Conventions: [add-translation](catalog/frontend/add-translation/SKILL.md) ·
[add-analytics-event](catalog/frontend/add-analytics-event/SKILL.md) ·
[add-testids](catalog/frontend/add-testids/SKILL.md) ·
[add-feature-flag](catalog/frontend/add-feature-flag/SKILL.md)
&nbsp;|&nbsp; Testing: [write-unit-tests](catalog/frontend/write-unit-tests/SKILL.md) ·
[write-e2e-test](catalog/frontend/write-e2e-test/SKILL.md) ·
[fix-flaky-test](catalog/frontend/fix-flaky-test/SKILL.md)
&nbsp;|&nbsp; Maintenance: [debug-ui-bug](catalog/frontend/debug-ui-bug/SKILL.md) ·
[perf-audit](catalog/frontend/perf-audit/SKILL.md) ·
[a11y-audit](catalog/frontend/a11y-audit/SKILL.md) ·
[upgrade-deps](catalog/frontend/upgrade-deps/SKILL.md) ·
[refactor-component](catalog/frontend/refactor-component/SKILL.md)
&nbsp;|&nbsp; UI stack: [theming](catalog/frontend/theming/SKILL.md) ·
[styled-components](catalog/frontend/styled-components/SKILL.md) ·
[antd](catalog/frontend/antd/SKILL.md)

Library conventions — State & data: [tanstack-query](catalog/frontend/tanstack-query/SKILL.md) ·
[zustand-store](catalog/frontend/zustand-store/SKILL.md) ·
[redux-toolkit](catalog/frontend/redux-toolkit/SKILL.md)
&nbsp;|&nbsp; Validation: [zod-schemas](catalog/frontend/zod-schemas/SKILL.md)
&nbsp;|&nbsp; Testing: [playwright-e2e](catalog/frontend/playwright-e2e/SKILL.md) ·
[vitest-unit](catalog/frontend/vitest-unit/SKILL.md) ·
[jest-unit](catalog/frontend/jest-unit/SKILL.md) ·
[testing-library-react](catalog/frontend/testing-library-react/SKILL.md)
&nbsp;|&nbsp; Routing: [react-router-v6](catalog/frontend/react-router-v6/SKILL.md)
&nbsp;|&nbsp; Utilities: [lodash-utils](catalog/frontend/lodash-utils/SKILL.md) ·
[date-fns-utils](catalog/frontend/date-fns-utils/SKILL.md)

## The core ideas

1. **Catalog broad, install narrow** — this repo costs nothing; skills
   installed in a work repo cost context every session. Cap at 5–8.
2. **Skills describe reality** — for existing repos, derive from the actual
   code; a skill that contradicts the repo makes AI output worse.
3. **Machine-checkable Definition of Done** — CLAUDE.md carries it, skills
   end with it, pre-push AI review uses it as the rubric, humans review only
   logic and product fit.
4. **Wrong twice → into the file** — corrections go into CLAUDE.md/skills the
   same day, never into individual prompting.
