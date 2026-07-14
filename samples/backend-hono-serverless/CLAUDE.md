# orders-service

Node 20 API on AWS Lambda. Hono for routing, Serverless Framework **v3 (3.40)**
for infra/deploys, TypeScript, Vitest, pnpm.

## Commands

- `pnpm install` — install deps
- `pnpm dev` — local API via serverless-offline at http://localhost:3000
- `pnpm test` — vitest run (watch: `pnpm test:watch`)
- `pnpm lint` / `pnpm lint:fix`
- `pnpm typecheck` — tsc --noEmit
- `pnpm deploy:dev` — `serverless deploy --stage dev`. **Never deploy staging/prod
  yourself — follow the `deploy` skill.**

## Architecture

- `src/app.ts` — the single Hono app; all routers registered here
- `src/routes/<resource>.ts` — one Hono router per resource (`orders.ts`, `customers.ts`)
- `src/handlers/api.ts` — Lambda entry: `export const handler = handle(app)`
  (`hono/aws-lambda` adapter). One catch-all `api` function handles all HTTP routes.
- `src/handlers/jobs/` — scheduled/queue Lambdas, one file per function, each
  declared separately in `serverless.yml`
- `src/lib/` — shared code: `db.ts` (DynamoDB DocumentClient), `logger.ts` (pino),
  `errors.ts`, `env.ts` (zod-parsed environment)
- `serverless.yml` — functions + resources. Plugins: `serverless-esbuild`,
  `serverless-offline`.

## Conventions

- **Every route that accepts input** uses `zValidator` (`@hono/zod-validator`)
  with a zod schema defined in the same file.
- Errors: throw `HTTPException` from `hono/http-exception`; the global `onError`
  in `app.ts` formats all errors as `{ error: { code, message } }`. Never return
  ad-hoc error shapes.
- Logging: use the pino logger from `src/lib/logger.ts`. No `console.log`.
- New env var = three places: `serverless.yml` `provider.environment`,
  `src/lib/env.ts` schema, and `.env.example`.
- Tests live next to the code (`orders.ts` → `orders.test.ts`). Route tests call
  `app.request(...)` directly — no HTTP server, no mocking of Hono itself.
- Serverless Framework is **v3** — do not use v4-only config or suggest
  upgrading; the version is pinned deliberately.

## Gotchas

- `serverless-offline` does not hot-reload `serverless.yml` changes — restart `pnpm dev`.
- Tests that touch DynamoDB need local Dynamo running: `docker compose up -d`.
- esbuild bundles per-function: new top-level runtime deps must NOT be in
  `devDependencies` or they silently vanish from the bundle.
