# Authoring & evolving skills

How to write a skill that a team can trust and evolve — and the verification
contract every skill states so its trust is explicit, not implied.

## The verification contract (every skill states this)

A skill is trustable when four things are explicit. Most of the catalog
already carries the middle two (a `## Verify` section + "pitfalls" / "review
catches"); this just names all four uniformly:

- **Inputs** — what the skill needs to run (the ticket/ACs, an API contract,
  a design, a schema).
- **Outputs** — what "done" produces (the component + hook + tests, the
  endpoint + IAM + tests…).
- **Verify** — the commands/observations that prove it: `typecheck` · `test`
  · `build` · drive it in the running app. Never "looks right."
- **Failure modes** — what review and tests must catch. These are the
  skill's reason to exist; if AI gets them wrong silently, the skill failed.

See the canonical worked example at the end of
[feature-development/new-form](frontend/feature-development/new-form/SKILL.md#verification-contract).

**Rollout:** new skills MUST include a `## Verification contract` section.
Existing skills get one when next edited — not in a mechanical mass pass, which
would bloat every installed skill's context for little gain since Verify +
pitfalls already exist. Incremental, on touch.

## Three layers — the skill must not hard-code layer 2 or 3

```
Generic skill   → "use axios interceptors, normalize to one error shape"
     ↓
Org standard    → "our ApiError has { status, code }"        (CLAUDE.md / DoD)
     ↓
Repo adaptation → repo A retry=yes, repo B retry=no          ([adapt] markers)
```

A skill that hard-codes `retry = 2` is wrong for the repo that wants none.
Skills state the PRINCIPLE and the DECISION TO MAKE; the number is an
`[adapt]` the repo resolves. This is why the catalog never installs verbatim.

## House rules for writing a skill

1. **Altitude** — encode the multi-step procedure people get wrong, plus the
   judgment lint can't enforce. If eslint/tsc/prettier already catches it,
   don't write a skill line for it.
2. **`[adapt]` every repo-specific value** — libraries, paths, versions,
   thresholds, tool names. A skill that contradicts the repo makes AI worse
   than no skill.
3. **Cross-link, don't duplicate** — reference the sibling skill
   (`see http-client §1`) instead of repeating it; duplicated rules drift.
4. **Size** — a skill is read into context on every matching task. Keep it
   tight; a 400-line skill is a cost. Split or trim.
5. **End with Verify + Failure modes** — the contract above.
6. **Describe reality** — for existing repos, derive from the actual code;
   for greenfield, impose the template. State which numbers are house
   choices vs. hard rules.

## Task skills vs library skills

- **Task skill** — a procedure end-to-end (`new-page`, `debug-prod-error`).
- **Library skill** — house conventions for one library (`tanstack-query`,
  `antd`). Install only for libraries the repo actually uses; where
  alternatives exist (vitest/jest, zustand/redux) — exactly one.

They pair by design: `write-e2e-test` (procedure) + `playwright-e2e`
(library) install together. See the catalog READMEs.
