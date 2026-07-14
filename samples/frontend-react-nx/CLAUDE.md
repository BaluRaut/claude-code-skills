# acme-web (Nx workspace)

React 18 apps in an Nx monorepo. pnpm workspace, Vite builds, Vitest +
Testing Library, Storybook for the design system.

## Commands

Always go through nx — never run vite/vitest directly:

- `pnpm nx serve <app>` — dev server. Apps: `admin`, `storefront`, `dashboard`
- `pnpm nx test <project>` (watch: add `--watch`)
- `pnpm nx lint <project>`
- `pnpm nx affected -t lint test build` — run before pushing; only touches
  projects affected by your change
- `pnpm nx g @nx/react:component ...` — scaffolding (see the `new-component` skill)
- `pnpm nx storybook ui` — design-system playground

## Structure

- `apps/*` — deployable apps. Keep them **thin**: routing + page composition.
  Real logic and UI live in libs.
- `libs/ui` — design-system components (Button, Card, DataTable…). Every
  component here has a Storybook story.
- `libs/data-access` — React Query hooks + API clients. Components never call
  `fetch`/`axios` directly — they use hooks from here.
- `libs/util` — pure functions only, no React.
- Cross-project imports use aliases (`@acme/ui`, `@acme/data-access`,
  `@acme/util`) — never relative paths across project boundaries; nx lint
  enforces module boundaries.

## Conventions

- **Server state = React Query** (hooks in `libs/data-access`, keys in
  `libs/data-access/src/keys.ts`). Local UI state = `useState`/`useReducer`.
  No Redux, no new state libraries.
- Function components only. Props type is `interface <Component>Props`,
  component exported from the lib's `index.ts` barrel.
- Styling: design tokens from `libs/ui/src/tokens.ts` — no hard-coded hex
  colors or px font sizes in components.
- Tests: Testing Library; query by role/label, not test-id, unless there is no
  accessible handle. One `.spec.tsx` next to each component.
- Shared by 2+ apps → `libs/ui` (with a story). Used by one app → that app's
  `components/` folder. When in doubt, start app-local; promote later.

## Gotchas

- Stale build/test results → `pnpm nx reset` clears the nx cache.
- `nx affected` compares against `main` — rebase first or it over-reports.
- Storybook and the app dev server can't share a port; storybook runs on 4400.
