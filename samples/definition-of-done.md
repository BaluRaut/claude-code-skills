# Definition of Done — paste into each repo's CLAUDE.md (adapted)

The contract every change must satisfy before a human reviews it. This is what
turns "AI writes, developer reviews" from a slogan into a system: Claude
follows it while coding, the pre-push AI review uses it as the rubric, and the
human reviewer only handles what's left — logic and product sense.

Pick ONE tool per category and delete the rest. Integrating Sentry AND Datadog
AND FullStory AND SmartLook multiplies cost and noise for no benefit.

## Frontend DoD (React apps)

### i18n — no hardcoded user-facing strings
- Every user-visible string goes through the i18n layer: `t('orders.refund.title')`,
  never a literal. This includes aria-labels, placeholders, error toasts.
- New keys: add to `src/locales/en.json` in the feature's namespace; other
  locales get the English value + a `// TODO:translate` marker (or your
  translation-queue process).
- Key naming: `<feature>.<screen>.<element>`. Never reuse a key for a
  different meaning because the English text happens to match.

### Analytics (Amplitude)
- Events are NEVER tracked inline with string literals. All events live in a
  typed catalog: `src/analytics/events.ts` — name, properties, and a comment
  saying when it fires.
- Event naming: `<Object> <Action>` past tense — `Refund Submitted`,
  `Filter Applied`. Properties in snake_case.
- New user-facing action = decide explicitly: track it (add to catalog) or
  note "no tracking" in the PR description. Silence is not a decision.

### Error tracking & session replay (Sentry + one of FullStory/SmartLook)
- No swallowed errors: every `catch` either recovers meaningfully or reports
  via `src/lib/monitoring.ts` (wraps Sentry) with feature context — never a
  bare `catch (e) {}`.
- New top-level routes/pages get an error boundary.
- Session replay: mask PII by default — any new input/element showing user
  data gets the mask class/attribute your replay tool requires.

### Testability — data-testid for Playwright
- Every interactive element (button, input, link, row action) and every
  assertion target (headings, totals, status badges) gets
  `data-testid="<feature>-<element>[-<action>]"`: `refund-submit`,
  `orders-row-menu`, `cart-total`.
- Testids are stable API for the QA/Playwright layer: renaming one is a
  breaking change — call it out in the PR description.
- Playwright tests select by role/label first, testid when that's ambiguous —
  but the testid must EXIST either way; QA writes E2E tests against them.

### Tests
- Changed logic = changed/added unit tests, in the same PR. New user flow =
  a Playwright spec (or an explicit "E2E deferred: <ticket>" note).
- Tests are derived from the ticket's ACCEPTANCE CRITERIA, not from the
  implementation. Each AC maps to at least one test that would fail if the
  AC were violated. (This is the guard against AI writing tests that merely
  confirm its own code.)

## Backend DoD (serverless services)

- Structured logs (pino) with the request ID on every path that can fail;
  no console.log.
- Errors: report to Sentry/Datadog with route + input context (PII-scrubbed);
  4xx are not "errors" in monitoring, 5xx always are.
- New endpoint/job = metric or alarm decision made explicitly (latency, DLQ
  depth, error rate) — "no alarm needed" is written in the PR if so.
- Changed logic = unit tests from acceptance criteria, same PR. New endpoint =
  route tests (happy path + validation failure + one error case) minimum.

## How this gets ENFORCED (no human nagging)

1. **CLAUDE.md carries this section** → Claude applies it while writing code.
2. **Skills reference it** → new-component / new-endpoint end with a
   "DoD check" step listing these items.
3. **pre-push AI review uses it as the rubric** → add to the hook prompt:
   "Also flag violations of the Definition of Done section in CLAUDE.md:
   hardcoded user-facing strings, interactive elements without data-testid,
   untracked user actions, swallowed errors, changed logic without tests."
4. **PR-level `/code-review --comment`** catches what slipped through, in
   public, before the human reviewer spends time.
5. The human reviewer reviews: does the logic match the ticket, is the
   approach right, do the tests actually test the ACs. Not formatting, not
   missing testids — the machine already did that.
