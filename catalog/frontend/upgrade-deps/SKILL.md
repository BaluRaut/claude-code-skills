---
name: upgrade-deps
description: Upgrade dependencies safely — one major at a time, changelog-driven, verified by build + tests + a real app run.
---

# Upgrade dependencies

## 1. Scope the batch

- Patch/minor updates: batch them (`pnpm update` within ranges) — one PR
- **Major versions: ONE per PR.** A 12-package major bump that breaks is
  undebuggable; twelve small PRs are each 10 minutes
- Security advisories first (`pnpm audit`), then majors by staleness

## 2. Before touching anything (majors)

- Read the changelog/migration guide for every major — list the breaking
  changes that apply to THIS codebase (grep for the affected APIs)
- Nx workspace: prefer `nx migrate <pkg>` — it applies codemods
  [adapt: plain repos check for official codemods, e.g. react-codemod]
- Check peer-dependency ripples: does this major force other bumps? If yes,
  that defines the PR's scope — nothing more

## 3. Upgrade

```
pnpm add <pkg>@<version>   # or nx migrate; commit lockfile changes deliberately
```

Apply the migration-guide changes. No opportunistic refactors in the same PR —
reviewers must see only upgrade-related changes.

## 4. Verify (all levels)

1. `pnpm typecheck && pnpm lint && pnpm test` — full suite, not affected-only,
   for majors
2. `pnpm build` — prod build catches what dev mode tolerates
3. Run the app and exercise the features the package touches (upgrade the
   date library → open every date picker)
4. Diff the lockfile: only expected packages changed

## 5. PR description

Package, old → new version, breaking changes applied, what you exercised
manually. Renovate/Dependabot [if configured — adapt] handles 1–2; this skill
is the human/AI procedure for the majors they can't auto-merge.
