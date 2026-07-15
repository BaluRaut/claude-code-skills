---
name: fix-flaky-test
description: Diagnose and fix a flaky test by classifying the actual cause — never by adding retries, sleeps, or longer timeouts.
---

# Fix a flaky test

Banned "fixes": increasing timeouts, adding `waitForTimeout`/sleeps, adding
retries, skipping the test. These hide the cause and the flake returns.

## 1. Reproduce and measure

```
[vitest/playwright] --repeat-each=20 <the test>
```

Also run it alongside its file/suite — some flakes only appear with neighbors.
Record the failure rate; you'll need it to prove the fix.

## 2. Classify the cause (in likelihood order)

1. **Missing await / unawaited async** — assertion runs before the UI settled.
   Fix: `await` the action; web-first assertions; Testing Library `findBy*`
   instead of `getBy*` after async actions.
2. **Shared state between tests** — module-level caches, localStorage,
   QueryClient reused, DB rows. Fix: fresh instances per test; isolate data.
3. **Order dependence** — passes alone, fails after a neighbor. Fix: the
   neighbor leaks state; fix the leak, not this test.
4. **Time** — `Date.now`, timezones, debounce timers. Fix: fake timers,
   injected clock, TZ-pinned CI.
5. **Real network / race with mocks** — mock set up after the request fires.
   Fix: mocks before navigation/render; fail tests on unmocked requests.
6. **Animation/transition races** — element "visible" but still moving.
   Fix: disable animations in test config.

## 3. Prove it

Same repeat run: 0 failures in 20+ (run the suite, not just the test). State
the cause + evidence in the PR description — "fixed flake" without a cause
named means it isn't fixed.
