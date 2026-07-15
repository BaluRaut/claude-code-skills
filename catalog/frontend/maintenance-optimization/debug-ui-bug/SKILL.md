---
name: debug-ui-bug
description: Debug a UI bug systematically — reproduce, isolate the layer, find the cause, fix with a regression test. No fix without a reproduction.
---

# Debug a UI bug

Rule zero: **no fix without a reproduction.** A patch for a bug you can't
reproduce is a guess wearing a commit message.

## 1. Reproduce

- Get exact steps, browser, viewport, account/role from the report
  ([session replay tool — adapt] usually has the recording — start there)
- Reproduce locally; if it needs prod data, reproduce the DATA SHAPE locally
- Write down the minimal repro steps — they become the regression test

## 2. Isolate the layer

Work outside-in; check where reality diverges from expectation:

1. **API**: network tab — is the response wrong? → backend bug, file it there
2. **State**: React Query devtools / component state — right data, wrong
   state? → cache keys, staleness, invalidation
3. **Logic**: right state, wrong computed value? → the transform/selector
4. **Render**: right value, wrong pixels? → component/CSS layer

If it's a regression: `git bisect` with the repro beats reading diffs.

## 3. Fix the cause, not the symptom

A `?.` or a `key={Math.random()}` that silences the error is not a fix.
You should be able to state WHY the bug happened in one sentence — that
sentence goes in the PR description.

## 4. Regression test

Turn the minimal repro into a test that fails without the fix and passes with
it. Run it against the unfixed code once to prove it catches the bug.

## 5. Check the blast radius

Same pattern elsewhere? Grep for it — bugs come in families. Note siblings
in the PR (fix here or file tickets).
