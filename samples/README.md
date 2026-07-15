# Claude Code repo templates

**Start with [common-rules.md](common-rules.md)** — the stack-independent
baseline every repo must satisfy, plus how to bootstrap an existing repo
(derive from real code) vs. a new one (impose the template).

**Then [definition-of-done.md](definition-of-done.md)** — the per-change
contract (i18n, analytics, Sentry/replay, data-testid, tests) that makes
"AI writes, developer reviews" enforceable without human nagging.

**Skill menu: [../catalog/frontend/](../catalog/frontend/)** — 29 ready-made
frontend skills in 8 categories (task skills + library-convention skills) to
pick from per repo (adapt placeholders first; install 5–8, prune monthly).

Sample setups matching our stacks. To use one, copy the **contents** of the
folder into the target repo's root (CLAUDE.md at root, `.claude/` at root) and
commit — every dev on the repo picks it up automatically.

```
backend-hono-serverless/       → for the Node + Hono + Serverless Framework 3.40 services
  CLAUDE.md
  .claude/settings.json        → shared permission allowlist + prettier hook
  .claude/skills/new-endpoint/ → scaffold a new HTTP endpoint end-to-end
  .claude/skills/deploy/       → safe deploy procedure (stage rules, rollback)

frontend-react-nx/             → for React apps living in the nx monorepo
  CLAUDE.md
  .claude/settings.json
  .claude/skills/new-component/ → scaffold a component the "house way"

frontend-react/                → for standalone React apps (pnpm + Vite, NO nx)
  CLAUDE.md
  .claude/settings.json
  .claude/skills/new-component/
  .claude/skills/deploy/       → build → upload → invalidate → verify (S3/CloudFront placeholder)
```

Before committing, replace the placeholders:

- `acme` / `@acme` — your org/npm scope
- app names (`admin`, `storefront`, `dashboard`) and service name (`orders-service`)
- commands, if your package.json scripts differ

Rule of thumb for maintaining these: CLAUDE.md is for facts Claude needs on
**every** task (commands, structure, conventions). A skill is for a **multi-step
procedure** that people get wrong (new endpoint, deploy). Keep CLAUDE.md near
100 lines — prune it when it grows.
