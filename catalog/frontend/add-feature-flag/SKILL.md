---
name: add-feature-flag
description: Add a feature flag correctly — registration, gating, default-off, QA override, and the cleanup ticket everyone forgets.
---

# Add a feature flag

Flag system: [LaunchDarkly / internal flags module — adapt].

## 1. Register

- Name: `<team>_<feature>_<yyyymm>` e.g. `checkout_refunds_v2_202607` — the
  date makes stale flags visible.
- Add to the typed flags module [src/lib/flags.ts — adapt] with a comment:
  what it gates, owner, target removal date. No stringly-typed flag checks
  scattered in components.

## 2. Gate

- Gate at the highest sensible level (route or page section), not deep inside
  leaf components — fewer branches to clean up later.
- Both branches must work: flag OFF is the current behavior, untouched.
- Default **off** everywhere; enabled per-environment/cohort in the flag tool.

## 3. QA path

Document in the PR how to turn it on locally/staging ([override mechanism —
adapt]). QA tests both states.

## 4. The cleanup ticket — not optional

Create a Jira ticket now: "Remove flag <name>", linked to the feature ticket,
due after full rollout. A flag without a removal ticket is permanent debt.

## 5. Tests

Both branches covered: one test with flag on, one with flag off, on the
behavior the flag changes.
