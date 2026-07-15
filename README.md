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

## Frontend skill catalog (36 skills, 9 categories)

A menu — install 5–8 per repo, adapt placeholders first, prune monthly.

- **[Catalog & menu](catalog/frontend/README.md)**
- **[Usage playbook](catalog/frontend/USAGE.md)** — real cases, three repo
  conditions (legacy / modern monorepo / greenfield), the 4-week adoption loop

**Feature Development:** [new-page](catalog/frontend/feature-development/new-page/SKILL.md) ·
[new-form](catalog/frontend/feature-development/new-form/SKILL.md) ·
[add-feature-flag](catalog/frontend/feature-development/add-feature-flag/SKILL.md) ·
[add-translation](catalog/frontend/feature-development/add-translation/SKILL.md) ·
[add-analytics-event](catalog/frontend/feature-development/add-analytics-event/SKILL.md)

**State Management & Data Fetching:** [new-data-hook](catalog/frontend/state-management-data/new-data-hook/SKILL.md) ·
[tanstack-query](catalog/frontend/state-management-data/tanstack-query/SKILL.md) ·
[zustand-store](catalog/frontend/state-management-data/zustand-store/SKILL.md) ·
[redux-toolkit](catalog/frontend/state-management-data/redux-toolkit/SKILL.md) ·
[immer](catalog/frontend/state-management-data/immer/SKILL.md) ·
[http-client](catalog/frontend/state-management-data/http-client/SKILL.md)

**Styling & Design Systems:** [theming](catalog/frontend/styling-design-systems/theming/SKILL.md) ·
[styled-components](catalog/frontend/styling-design-systems/styled-components/SKILL.md) ·
[antd](catalog/frontend/styling-design-systems/antd/SKILL.md)

**Validation & Forms:** [zod-schemas](catalog/frontend/validation-forms/zod-schemas/SKILL.md)

**Testing & QA:** [write-unit-tests](catalog/frontend/testing-qa/write-unit-tests/SKILL.md) ·
[write-e2e-test](catalog/frontend/testing-qa/write-e2e-test/SKILL.md) ·
[fix-flaky-test](catalog/frontend/testing-qa/fix-flaky-test/SKILL.md) ·
[add-testids](catalog/frontend/testing-qa/add-testids/SKILL.md) ·
[playwright-e2e](catalog/frontend/testing-qa/playwright-e2e/SKILL.md) ·
[vitest-unit](catalog/frontend/testing-qa/vitest-unit/SKILL.md) ·
[jest-unit](catalog/frontend/testing-qa/jest-unit/SKILL.md) ·
[testing-library-react](catalog/frontend/testing-qa/testing-library-react/SKILL.md) ·
[msw-mocking](catalog/frontend/testing-qa/msw-mocking/SKILL.md)

**Framework Core:** [react-router-v6](catalog/frontend/framework-core/react-router-v6/SKILL.md) ·
[react-effects](catalog/frontend/framework-core/react-effects/SKILL.md)

**Utilities & Performance:** [lodash-utils](catalog/frontend/utilities-performance/lodash-utils/SKILL.md) ·
[date-fns-utils](catalog/frontend/utilities-performance/date-fns-utils/SKILL.md)

**Maintenance & Optimization:** [debug-ui-bug](catalog/frontend/maintenance-optimization/debug-ui-bug/SKILL.md) ·
[perf-audit](catalog/frontend/maintenance-optimization/perf-audit/SKILL.md) ·
[a11y-audit](catalog/frontend/maintenance-optimization/a11y-audit/SKILL.md) ·
[upgrade-deps](catalog/frontend/maintenance-optimization/upgrade-deps/SKILL.md) ·
[refactor-component](catalog/frontend/maintenance-optimization/refactor-component/SKILL.md)

**Security & Reliability:** [error-monitoring](catalog/frontend/security-reliability/error-monitoring/SKILL.md) ·
[frontend-security](catalog/frontend/security-reliability/frontend-security/SKILL.md) ·
[auth-patterns](catalog/frontend/security-reliability/auth-patterns/SKILL.md)

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
