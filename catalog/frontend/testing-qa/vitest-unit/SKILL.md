---
name: vitest-unit
description: Vitest conventions — config shared with Vite, environment choice, vi.mock hoisting rules, fake timers, snapshot policy.
---

# Vitest

For Vite-based repos [and nx — adapt]. If the repo runs Jest, use the
jest-unit skill instead — one runner per repo, never both.

## 1. Config [vitest.config.ts / vite.config.ts test block — adapt]

- Reuse the app's Vite config — aliases (`@/`) and plugins come free;
  a separate diverging test config is a maintenance bug
- `environment: 'jsdom'` [or 'happy-dom' — adapt] for component tests;
  pure-logic packages use 'node' (faster)
- `globals: true` [house choice — adapt] — pick one style repo-wide;
  mixed explicit/global imports across files fails review
- `setupFiles`: one shared setup (jest-dom matchers, MSW server, cleanup)

## 2. Mocking — the hoisting rules

- `vi.mock('./module')` is HOISTED — factory can't touch outer variables;
  use `vi.hoisted(() => ...)` for shared mock refs
- Mock at boundaries (API client, analytics) — never internal modules of
  the unit under test; that's testing implementation (see write-unit-tests)
- Partial mock: `importOriginal` inside the factory —
  `{ ...(await importOriginal()), track: vi.fn() }`
- `vi.restoreAllMocks()` in afterEach [or config `restoreMocks: true`]

## 3. Time & async

- `vi.useFakeTimers()` for debounce/polling logic — and ALWAYS
  `vi.useRealTimers()` in afterEach, or later tests hang mysteriously
- Timers + user-event need `advanceTimers` wired
  (`userEvent.setup({ advanceTimers: vi.advanceTimersByTime })`)
- Unhandled async: `expect(promise).rejects.toThrow(...)` — awaited, not
  fire-and-forget

## 4. Snapshot policy

Inline snapshots (`toMatchInlineSnapshot`) for small serializable values
only. Full-component `toMatchSnapshot` is banned — nobody reviews 200-line
snapshot diffs; assert the specific thing instead.

## 5. Running

- Single file: `pnpm vitest run <path>`; watch mode for the file you're on
- Flaky suspicion: `--repeat-each` per fix-flaky-test — never `retry` in
  config to paper over it
