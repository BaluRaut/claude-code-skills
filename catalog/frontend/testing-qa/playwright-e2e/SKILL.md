---
name: playwright-e2e
description: Playwright conventions — config, fixtures for auth/data, selector strategy, web-first assertions, parallel-safe tests, trace-based CI debugging.
---

# Playwright

Library conventions; the task-level procedure for writing a spec is the
`write-e2e-test` skill.

## 1. Config [playwright.config.ts — adapt]

- `baseURL` from env — specs never hardcode hosts
- `trace: 'on-first-retry'`, `screenshot: 'only-on-failure'` — CI failures
  come with evidence attached
- Projects: chromium always; [webkit/firefox/mobile — house choice]
- `fullyParallel: true` is the goal — it forces test isolation (below)
- Web-first assertion timeout tuned once globally — specs never override
  timeouts per-assertion to "fix" flakes (see fix-flaky-test)

## 2. Fixtures — auth and data, not copy-paste beforeEach

- **Auth**: `storageState` per role, produced once in [global setup — adapt]:
  tests start logged in; the login FLOW is tested by exactly one spec
- **Data**: custom fixtures create what a test owns and clean up after —
  `test.extend<{ order: Order }>` beats 30 lines of setup per spec
- Parallel-safety rule: a test may only touch data it created — shared
  seeded accounts + `fullyParallel` = the flake factory

## 3. Selectors (contract with add-testids)

Priority: `getByRole`/`getByLabel` → `getByTestId` when ambiguous. Banned:
CSS/class selectors, XPath, `.nth(3)` positional selection. If a page object
layer exists [e2e/pages — adapt], selectors live there, not in specs.

## 4. Assertions & waiting

- `await expect(locator).toBeVisible()/toContainText()` — web-first retries
  built in
- `waitForTimeout` is banned; `waitForLoadState('networkidle')` is a smell —
  assert on what the USER sees instead
- Assert outcomes, not network responses (unless the API call IS the
  contract, e.g. analytics — then `page.waitForRequest` narrowly)

## 5. Suites & CI

- Tags: `@smoke` (minutes, every PR) vs full regression [schedule — adapt]
- A failed CI run is debugged from the trace viewer
  (`npx playwright show-trace`) — not by re-running until green; retries in
  CI are a flake alarm, not a fix

## 6. Verify

New/changed specs: `--repeat-each=5` locally, plus one headed run
(`--headed`) to see the flow do what the ticket says.
