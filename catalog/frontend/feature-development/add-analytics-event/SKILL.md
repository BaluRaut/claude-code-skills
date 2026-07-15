---
name: add-analytics-event
description: Add an Amplitude event the disciplined way — typed catalog entry, naming convention, correct firing moment, and a test that it fires.
---

# Add an analytics event

## 1. Check the catalog first

All events live in [src/analytics/events.ts — adapt]. Search it — an event
for this action may already exist; reuse beats near-duplicates
(`Refund Submitted` vs `Submit Refund` pollutes every dashboard).

## 2. Define in the catalog (never inline)

```ts
// events.ts
export const events = {
  refundSubmitted: (props: { order_value_cents: number; reason: string }) =>
    track('Refund Submitted', props),
}
```

- Name: `<Object> <Action>`, past tense — `Refund Submitted`, `Filter Applied`
- Properties: snake_case; no PII (no emails, names, free-text user input)
- Comment: when exactly it fires

## 3. Fire at the right moment

- On **outcome**, not intent: after the mutation succeeds, not on button click
  (unless intent is the thing being measured — then name it `... Clicked`)
- Once per action — guard against firing in re-renders/effects
- Never block or break UX: analytics failures are swallowed by the client
  wrapper, not by your code

## 4. Test + verify

- Unit: mock the analytics client, assert the event fires with correct props
  on the success path, and does NOT fire on failure
- Dev: confirm in the network tab / Amplitude debugger it arrives once
- PR description: state the new event name so product/analytics see it
