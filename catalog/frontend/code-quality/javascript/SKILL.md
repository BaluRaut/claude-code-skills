---
name: javascript
description: Modern JavaScript house rules — async/await discipline, parallelism, nullish vs truthy, immutability defaults, error objects, and the everyday traps.
---

# JavaScript conventions

The language-level rules under everything else. Lint enforces syntax; this
skill encodes the JUDGMENT lint can't.

## 1. Async discipline (most JS bugs live here)

- `async/await` over `.then()` chains; no mixing both in one flow
- **Independent awaits run in parallel**:
  `const [user, orders] = await Promise.all([getUser(id), getOrders(id)])` —
  sequential awaits of independent calls is the #1 silent latency bug
- `await` in a loop only when iterations DEPEND on each other; otherwise
  map to promises + `Promise.all` (bounded: chunk if N is large [adapt])
- No floating promises: every promise is awaited, returned, or explicitly
  `void`-ed with a comment — `no-floating-promises` lint on
- `Promise.allSettled` when partial failure is acceptable — then HANDLE the
  rejected entries; ignoring them is a swallowed error (error-monitoring)

## 2. Nullish vs truthy (the 0/'' trap)

- `??` for defaults, not `||`: `count || 10` turns a legitimate `0` into
  `10` — classic. `||` only when falsy-means-absent is truly intended
- `?.` for optional access — but 3+ chained `?.` means the TYPE is wrong
  (see typescript): fix the shape, don't tunnel through it
- `=== `/`!==` always; comparing to `null` with `== null` is the one
  allowed exception (catches undefined too) [house choice — adapt]

## 3. Immutability by default

- `const` everywhere; `let` is a signal to read carefully; `var` never
- Don't mutate params or shared objects — return new values (spread /
  `toSorted`/`toReversed` over `sort`/`reverse` [runtime target — adapt]);
  deep updates = immer (see immer skill)

## 4. Errors are Error objects

- `throw new Error('...')` (or app subclasses) — never strings/objects;
  stack traces and monitoring depend on it
- `catch (e)`: `e` is `unknown` — narrow before using (typescript §4);
  rethrow with `{ cause: e }` when wrapping, so the chain survives

## 5. Leaks & closures (the slow bugs)

- Everything started gets stopped: listeners/timers/observers/sockets have
  a cleanup path — in React that's effect cleanup (react-effects §2)
- Module-level caches are UNBOUNDED by default → give them eviction (LRU,
  TTL) or `WeakMap`/`WeakRef` keyed by the object whose lifetime you mean
- Stale closures: a long-lived callback (timer, subscription, debounce)
  captures the variables from WHEN IT WAS CREATED — re-create it when deps
  change or read through a ref; "why does it log the old value" is this
- Closures capturing large objects keep them alive — extract just the
  fields the callback needs

## 6. Everyday judgment

- Array methods over loops when it reads better — but a `for...of` beats a
  4-level method chain; readability wins, not dogma
- Named constants for magic numbers/strings used more than once or
  non-obvious once
- Intl for dates/numbers/currency (date-fns-utils) — never string math
- Modules: named exports [house choice — adapt]; no side effects at import
  time beyond constants (breaks tree-shaking and test isolation)
