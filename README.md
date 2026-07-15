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

## Frontend skill catalog (43 skills, 11 categories)

A menu — install 5–8 per repo, adapt placeholders first, prune monthly.

- **[Catalog & menu](catalog/frontend/README.md)**
- **[Usage playbook](catalog/frontend/USAGE.md)** — real cases, three repo
  conditions (legacy / modern monorepo / greenfield), the 4-week adoption loop

**Feature Development:** [new-page](catalog/frontend/feature-development/new-page/SKILL.md) ·
[new-form](catalog/frontend/feature-development/new-form/SKILL.md) ·
[add-feature-flag](catalog/frontend/feature-development/add-feature-flag/SKILL.md) ·
[add-translation](catalog/frontend/feature-development/add-translation/SKILL.md)

**Product Analytics:** [add-analytics-event](catalog/frontend/product-analytics/add-analytics-event/SKILL.md) ·
[amplitude](catalog/frontend/product-analytics/amplitude/SKILL.md)

**State Management & Data Fetching:** [new-data-hook](catalog/frontend/state-management-data/new-data-hook/SKILL.md) ·
[tanstack-query](catalog/frontend/state-management-data/tanstack-query/SKILL.md) ·
[zustand-store](catalog/frontend/state-management-data/zustand-store/SKILL.md) ·
[redux-toolkit](catalog/frontend/state-management-data/redux-toolkit/SKILL.md) ·
[immer](catalog/frontend/state-management-data/immer/SKILL.md) ·
[http-client](catalog/frontend/state-management-data/http-client/SKILL.md) ·
[axios](catalog/frontend/state-management-data/axios/SKILL.md)

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
[refactor-component](catalog/frontend/maintenance-optimization/refactor-component/SKILL.md) ·
[component-review](catalog/frontend/maintenance-optimization/component-review/SKILL.md)

**Code Quality & Design:** [javascript](catalog/frontend/code-quality/javascript/SKILL.md) ·
[typescript](catalog/frontend/code-quality/typescript/SKILL.md) ·
[code-design-solid](catalog/frontend/code-quality/code-design-solid/SKILL.md)

**Security & Reliability:** [error-monitoring](catalog/frontend/security-reliability/error-monitoring/SKILL.md) ·
[frontend-security](catalog/frontend/security-reliability/frontend-security/SKILL.md) ·
[auth-patterns](catalog/frontend/security-reliability/auth-patterns/SKILL.md) ·
[datadog-rum](catalog/frontend/security-reliability/datadog-rum/SKILL.md)

## Backend skill catalog (22 skills, 8 categories)

Node serverless on AWS — Hono + Serverless Framework v3. Same rules:
install only for infrastructure the service actually uses.

- **[Catalog & menu](catalog/backend/README.md)**

**Service Patterns:** [hono-patterns](catalog/backend/service-patterns/hono-patterns/SKILL.md) ·
[hono-middleware](catalog/backend/service-patterns/hono-middleware/SKILL.md) ·
[api-design](catalog/backend/service-patterns/api-design/SKILL.md) ·
[serverless-v3-config](catalog/backend/service-patterns/serverless-v3-config/SKILL.md) ·
[lambda-patterns](catalog/backend/service-patterns/lambda-patterns/SKILL.md) ·
[aws-sdk-v3](catalog/backend/service-patterns/aws-sdk-v3/SKILL.md)

**Data & Storage:** [dynamodb](catalog/backend/data-storage/dynamodb/SKILL.md) ·
[rds-postgres](catalog/backend/data-storage/rds-postgres/SKILL.md) ·
[s3](catalog/backend/data-storage/s3/SKILL.md)

**Messaging & Events:** [sqs](catalog/backend/messaging-events/sqs/SKILL.md) ·
[eventbridge](catalog/backend/messaging-events/eventbridge/SKILL.md)

**Security & Config:** [api-security](catalog/backend/security-config/api-security/SKILL.md) ·
[secrets-config](catalog/backend/security-config/secrets-config/SKILL.md)

**Infra & Delivery:** [cloudfront](catalog/backend/infra-delivery/cloudfront/SKILL.md) ·
[vpc-networking](catalog/backend/infra-delivery/vpc-networking/SKILL.md) ·
[containers-eks-ecr](catalog/backend/infra-delivery/containers-eks-ecr/SKILL.md)

**Observability:** [logging-observability](catalog/backend/observability/logging-observability/SKILL.md) ·
[debug-prod-error](catalog/backend/observability/debug-prod-error/SKILL.md) ·
[sentry-node](catalog/backend/observability/sentry-node/SKILL.md) ·
[datadog-apm](catalog/backend/observability/datadog-apm/SKILL.md)

**Code Quality:** [code-design-solid](catalog/backend/code-quality/code-design-solid/SKILL.md)

**Testing:** [node-testing](catalog/backend/testing/node-testing/SKILL.md)

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
