# Common rules — every repo, new or existing

The stack-independent baseline. A repo is "Claude-ready" when all of this holds.
For **existing** repos: derive the content from the actual code (run `/init`,
then curate). For **new** repos: impose the matching template from this folder.

## 1. Required files (checked into git)

- [ ] `CLAUDE.md` at repo root — under ~100 lines, reviewed by a senior dev
- [ ] `.claude/settings.json` — permission allowlist + formatter hook + deny list
- [ ] `.claude/skills/` — max 5–8 skills, each documenting a real multi-step procedure
- [ ] `.env.example` — every env var the code reads, no real values

## 2. Required CLAUDE.md sections (content must match reality, not aspiration)

- [ ] **Commands** — install, dev, test (and how to run a single test), lint, typecheck, build
- [ ] **Structure** — where things live, in 5–10 lines; include "where does X go" for the most common X (component, endpoint, helper)
- [ ] **Conventions** — only rules the code actually follows today. If the repo is inconsistent, state the *target* convention explicitly: "mixed today; all NEW code uses X"
- [ ] **Gotchas** — the 3–5 things that waste an hour of a new dev's time
- [ ] Nothing generic ("write clean code", "follow best practices") — if a linter enforces it or Claude already knows it, delete the line

## 3. Safety rules (same in every repo)

- [ ] Deny list covers the destructive stuff: prod deploy, `serverless remove` /
      infra teardown, force-push, dropping/along migrations, deleting data
- [ ] CLAUDE.md states what needs explicit human confirmation (prod anything)
- [ ] Secrets: never in code, never in CLAUDE.md, never in skills — document
      *where* they live (SSM, .env from 1Password…), not the values
- [ ] Generated files (lockfiles, codegen output, snapshots) named as do-not-hand-edit

## 4. Verification rule (the one that matters most)

- [ ] Every skill ends with a "verify" step (typecheck + lint + test at minimum)
- [ ] CLAUDE.md names the single command that must pass before a PR:
      e.g. `pnpm verify` or `pnpm nx affected -t lint test build`
- [ ] If that one command doesn't exist as a script yet, create it — one
      command beats four that people (and Claude) run inconsistently

## 5. Process rules (people, not files)

- [ ] One named owner per repo for CLAUDE.md + skills
- [ ] Claude did something wrong twice → the fix goes into CLAUDE.md or a
      skill, not into individual prompting
- [ ] Changes to `.claude/` go through normal PR review — treat instructions
      to Claude with the same care as code, they run on every dev's machine
- [ ] Monthly prune: delete skills nobody invoked, cut CLAUDE.md lines that
      stopped being true
- [ ] Skills are written from how the team ACTUALLY works. If the skill is
      meant to *change* how the team works, agree on that in review first —
      then the skill enforces the new way for everyone, including Claude

## How to bootstrap an existing repo (order matters)

1. `/init` → curate CLAUDE.md down to what's true and useful
2. Add `.claude/settings.json` — start with the deny list, then run
   `/fewer-permission-prompts` after a week of real use for the allowlist
3. Write the FIRST skill only when a procedure has been done wrong twice
4. Ask Claude to draft it *from the repo's own code*, review it like a PR
