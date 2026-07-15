---
name: write-unit-tests
description: Write unit tests for changed code, derived from acceptance criteria — not from the implementation. The guard against tests that merely confirm the code.
---

# Write unit tests

## 1. Start from the acceptance criteria, not the code

Read the ticket's ACs (or ask the dev for the intended behavior). Build the
mapping BEFORE reading the implementation deeply:

| AC / behavior | Test |
|---|---|
| Refund over order total is rejected | `rejects amount exceeding order total` |
| Successful refund shows confirmation | `shows confirmation on success` |

Every AC gets at least one test **that would fail if the AC were violated**.
If you can't write that test, the AC is untestable — flag it to the dev.

## 2. Add the edges the ACs don't mention

Empty input, zero/negative numbers, error responses, loading interruptions,
permission-denied. One test per edge that the code claims to handle.

## 3. House patterns

- Testing Library: query by role/label first, testid when ambiguous —
  never by class or DOM structure
- Test behavior through the public surface (props in, rendered output +
  callbacks out) — not internal state, not "was this function called" unless
  the call IS the contract (analytics, mutations)
- Mock at the boundary ([MSW / api mocks — adapt]), not internal modules
- One behavior per test; the test name states the behavior, not the method name

## 4. Prove the tests can fail

For at least the critical ACs: temporarily break the code (invert the
condition), confirm the test fails, restore. A test suite that can't fail is
decoration.

## 5. Verify

Repo test command green; coverage on the changed files meaningfully touched
(don't chase a number — chase the AC table being complete).
