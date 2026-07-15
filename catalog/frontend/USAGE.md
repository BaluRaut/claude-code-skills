# Using the skill catalog — real cases, real repo conditions

How the catalog's skills behave in practice, and which subset fits which
kind of repo. Read this before installing anything.

Library skills (tanstack-query, antd, vitest-unit…) follow one extra rule:
install ONLY for libraries the repo actually uses, at the version the repo
actually has — and where alternatives exist (vitest/jest, zustand/redux),
exactly one.

## How a skill actually fires

Two ways, both automatic once installed in `.claude/skills/`:

1. **Explicit**: dev types `/new-page refunds screen` — the skill loads and
   Claude follows its steps.
2. **Implicit**: dev types "add a refunds screen with a list and a create
   form" — Claude matches the task against installed skills' descriptions and
   pulls in `new-page`, `new-data-hook`, `new-form` on its own.

This is why descriptions matter and why you DON'T install all 38: every
installed skill is a candidate on every task. Ten well-chosen skills get
matched correctly; thirty make matching mushy.

---

## Repo condition 1: the legacy app

*React 17-ish, antd v4, styled-components v5, strings hardcoded, ~10% test
coverage, no testids, no i18n discipline. Two devs maintain it part-time.*

**Install (5):** `debug-ui-bug`, `write-unit-tests`, `add-testids`,
`add-translation`, `antd` (pinned to **v4** — the skill's v5 advice must be
swapped for less-variable theming when you adapt it).

**Don't install:** `new-page`/`new-form`/`new-data-hook` — they prescribe
patterns (React Query, zod) this repo doesn't have. A skill that contradicts
the repo teaches Claude to write foreign code into it.

**Real case:** *Ticket: "Order total shows ₹0 after applying coupon."*
Dev: "debug this — order total shows 0 after coupon applied, here's the repro".
`debug-ui-bug` drives it: reproduce → network tab shows API returns the right
total → state layer shows a stale memo dropping the coupon → one-sentence
cause in the PR → `write-unit-tests` turns the repro into a regression test.
Result: a fix PR with a test, from a dev who didn't know this code area.

**The trap here:** don't try to modernize via skills ("rewrite this screen
with React Query"). Legacy repos get *stabilizing* skills first; migration is
a decision for you + ADR, not a side effect of a skill.

## Repo condition 2: the main product (nx monorepo, modern)

*React 18, antd v5, React Query, i18n wired, styled-components v6, decent
tests. Most feature work happens here.*

**Install (7–8):** `new-page`, `new-data-hook`, `new-form`,
`add-analytics-event`, `add-translation`, `write-unit-tests`, `theming`,
`antd` (pinned v5). Plus `new-component` from the nx sample. E2E skills go in
when QA joins the repo.

**Real case:** *Ticket PROJ-482: "Customers can request a refund from order
detail."* The full AI-enabled-developer loop:

1. `/start-task PROJ-482` → ticket completed with 3 ACs, subtasks created,
   branch `feature/PROJ-482-refund-request` made, **plan presented — dev
   approves it** (decision point #1)
2. Dev: "implement subtask 1, the refund form on order detail" → Claude
   auto-matches `new-form` + `new-data-hook`: zod schema, mutation hook with
   invalidation, submit states, i18n keys, testids — because the skills
   demand them, not because anyone remembered
3. Dev: "now the tests" → `write-unit-tests` builds the AC→test table; dev
   checks the TABLE, not every line (decision point #2: do these tests
   actually verify the ACs?)
4. `git push` → pre-push hook prints 2 findings (a swallowed catch, a missing
   empty state) → "fix both" → push clean
5. `/open-pr` → self-review, PR with template, Jira moves to In Review
6. Human reviewer reads a PR that's already mechanically clean — reviews
   ONLY: is this the right UX, do the tests match the ticket

Dev wrote ~0 lines and made ~4 judgment calls. That's the model working.

## Repo condition 3: greenfield app

*New repo, starting next sprint.*

Impose the full standard from day one — this is the ONE case where templates
go in verbatim: copy `samples/frontend-react[-nx]`, the DoD into CLAUDE.md,
and install the build set (`new-page`, `new-data-hook`, `new-form`,
`theming`, `antd`, `write-unit-tests`). Every screen is then born with i18n,
testids, analytics, and tests — retrofitting costs 10x.

---

## Case → skill quick map

| Situation | Reach for |
|---|---|
| "Build the X screen" | new-page → new-data-hook → new-form |
| "Users report X is broken" | debug-ui-bug (+ write-unit-tests for the regression) |
| "CI is red randomly again" | fix-flaky-test |
| "This page feels slow" | perf-audit (numbers first, then fixes) |
| "QA can't automate the orders screen" | add-testids sweep |
| "Accessibility audit next month" | a11y-audit per key page, worst first |
| "antd 5.12 → 5.22" / any major bump | upgrade-deps (+ antd skill's visual pass) |
| "Dark mode looks broken on X" | theming (usually a raw hex hiding somewhere) |
| "This 900-line component again…" | refactor-component (tests green FIRST) |
| "Track how many people use filters" | add-analytics-event (catalog first — it may exist) |
| Marketing wants German | add-translation discipline BEFORE the locale lands |

## Making the rules stick (your 4-week loop per repo)

1. **Week 0** — adapt placeholders with the repo's senior dev (30 min: which
   form lib, antd version, i18n lib, where keys live). Wrong placeholders =
   Claude confidently doing the wrong thing.
2. **Weeks 1–3** — devs work normally. One rule for them: *if Claude produced
   code that violated a convention, say which skill/CLAUDE.md line was missing
   or wrong in the team channel.* Fix the file same day.
3. **Week 4** — prune: each dev names skills they used / never used. Delete
   the unused ones from the repo (they stay in this catalog). Promote
   anything every repo kept into the shared plugin.

**Signs it's working:** PR review comments shift from "missing testid /
hardcoded string" to product questions; pre-push findings per push trend
down; juniors ship in unfamiliar code areas.

**Signs a skill is broken:** devs bypass it ("it always suggests the wrong
form lib") — that's a placeholder you didn't adapt. Fix or delete same week;
a distrusted skill poisons trust in all of them.
