---
name: jest-unit
description: Jest conventions — TS/alias config, module mock patterns, fake timers, snapshot policy. For repos standardized on Jest.
---

# Jest

For repos already on Jest [version — adapt]. Vite repos should use Vitest
(see vitest-unit); don't migrate runners as a side effect of a feature PR —
that's its own decision.

## 1. Config [jest.config.ts — adapt]

- `moduleNameMapper` MUST mirror tsconfig paths (`@/` → `src/`) and stub
  assets/styles (`identity-obj-proxy` for CSS modules) — most "jest can't
  find module" issues live here
- `testEnvironment: 'jsdom'` for components, `'node'` for pure logic
- Transform: [ts-jest / babel-jest / @swc/jest — adapt to what the repo
  uses; don't mix]
- One `setupFilesAfterEnv` for jest-dom matchers, MSW, global cleanup

## 2. Mocking

- `jest.mock('./module')` is hoisted — same discipline as everywhere:
  mock BOUNDARIES (API client, analytics), not internals of the unit
  under test (see write-unit-tests)
- Partial: `jest.requireActual` spread + override the one function
- `clearMocks: true` + `restoreMocks: true` in config — manual
  `jest.clearAllMocks()` sprinkled in files means config is wrong
- Reusable manual mocks in `__mocks__/` next to the module — one mock
  definition, not five copies

## 3. Time & async

- `jest.useFakeTimers()` for debounce/interval logic; restore in afterEach
- user-event with fake timers needs
  `userEvent.setup({ advanceTimers: jest.advanceTimersByTime })`
- Async assertions awaited: `await expect(fn()).rejects.toThrow(...)`

## 4. Snapshot policy

Same as the house rule everywhere: small inline snapshots fine; whole-tree
`toMatchSnapshot` banned — assert the specific behavior instead.

## 5. Running

- One file: `pnpm jest <path>`; one test: `-t "name"`
- `--detectOpenHandles` when the run doesn't exit — usually an unclosed
  timer/socket from a test that mocked too little
