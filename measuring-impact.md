# Measuring impact

Successful builds prove the skills WORK. These metrics prove they're worth it.
Measure outcomes, not activity — and pair every velocity number with a quality
number, or you'll optimize speed by shipping bugs.

## The two headline metrics (if you track nothing else, track these)

| Metric | Meaning | Reads as |
|---|---|---|
| **Cycle time** (in-progress → merged) | how fast a story ships | down = faster |
| **Escaped defects** (bugs found after merge, per story, within 2 weeks) | did speed cost quality | flat or down = quality held |

Cycle time **down** + escaped defects **flat/down** = the system works.
Cycle time down + escaped defects **up** = you're shipping faster by shipping
worse — tighten the gates (DoD, pre-push review) before scaling.

## The full table (capture a 2-sprint baseline BEFORE the pilot)

| Metric | Where the number comes from | Cadence |
|---|---|---|
| Lead time (story → merge) | Jira created-date → GitHub PR merged-date | per story |
| Cycle time (In Progress → merge) | Jira status change → PR merged | per story |
| Time to implement | Jira In Progress → In Review duration | per story |
| Review comments per PR | GitHub API: review comment count | per PR |
| Review rounds / rework | commits pushed after first review; # review cycles | per PR |
| Time reviewing | review-requested → approved elapsed (proxy) | per PR |
| Escaped defects | Jira bugs linked to the story within 2 weeks of merge | per story |
| Coverage on new code | coverage tool, changed-files only (not whole-repo %) | per PR |

Most of this is already in Jira + GitHub — a small script over both APIs
produces the table. Don't hand-collect; if it's manual it won't survive.

## Baseline plan

1. **Weeks 0–4 (before skills):** run the script over the last ~2 sprints of
   the pilot repo. That's your baseline — no skills installed yet.
2. **Weeks 5–8 (skills installed, adapted to the repo):** same script, same
   repo. Compare.
3. Judge on the trend across ≥2 sprints, not any single PR — variance is high
   at the story level.

## The gaming traps (Goodhart's law is real)

- **"Fewer review comments"** is NOT a goal — it also happens when reviewers
  stop reading. Pair it with escaped defects.
- **"More PRs / higher velocity"** invites PR-splitting theater. Pair with
  lead time and defects.
- **Coverage %** invites assertion-free tests. The `write-unit-tests` skill's
  "prove the test can fail" rule is the real guard; coverage is a floor, not
  a target.
- Never rank individual developers on these — they measure the SYSTEM. The
  moment they're used to rank people, they stop being honest.

## Evidence already in hand

The `customers-app` pilot (career/skills-pilot) built CUS-101 from 21 skills:
strict typecheck, 10 AC-derived tests, and a production build all pass — and
the verify step caught a real integration bug (axios fetch-adapter under
jsdom) before any human review. That's the internal-consistency proof; this
table is how you prove the team-level payoff on a real repo.
