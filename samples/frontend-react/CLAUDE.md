# acme-portal (standalone React app)

Single React 18 app — pnpm, Vite, TypeScript, Vitest + Testing Library,
React Router, React Query. No monorepo tooling: plain package.json scripts.

## Commands

- `pnpm install` — install deps
- `pnpm dev` — Vite dev server at http://localhost:5173
- `pnpm test` — vitest run (watch: `pnpm test:watch`)
- `pnpm lint` / `pnpm lint:fix`
- `pnpm typecheck` — tsc --noEmit
- `pnpm build` — production build to `dist/` (run before finishing any change
  that touches build config or env usage)
- `pnpm preview` — serve the production build locally

## Structure

- `src/pages/` — one folder per route (`orders/`, `settings/`); pages compose,
  they don't contain business logic
- `src/components/` — shared components for this app, one folder per component
  (`component.tsx`, `component.spec.tsx`, optional `component.module.css`)
- `src/api/` — React Query hooks + the fetch client (`client.ts`). Components
  never call `fetch` directly — always through a hook from here. Query keys
  live in `src/api/keys.ts`.
- `src/lib/` — pure helpers, no React
- `src/router.tsx` — all routes declared here
- Path alias: `@/` → `src/` (configured in vite.config.ts and tsconfig)

## Conventions

- **Server state = React Query** (hooks in `src/api/`). Local UI state =
  `useState`/`useReducer`. No Redux, no new state libraries.
- Function components only. Props type is `interface <Component>Props`.
- Styling: CSS modules with variables from `src/styles/tokens.css` — no
  hard-coded hex colors or px font sizes in components.
- Env vars: `VITE_`-prefixed, accessed only through `src/lib/env.ts` (typed,
  validated at startup) — never `import.meta.env` scattered in components.
  New var also goes in `.env.example`.
- Tests: Testing Library next to the component; query by role/label, not
  test-id, unless there is no accessible handle.

## Gotchas

- Changing `vite.config.ts` or `.env*` requires restarting `pnpm dev`.
- `pnpm build` typechecks stricter than the dev server — a change can "work"
  in dev and still break the build; always build before finishing config work.
