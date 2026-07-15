---
name: write-e2e-test
description: Write a Playwright E2E spec for a user flow — selector strategy, data setup, web-first assertions, no sleeps.
---

# Write an E2E test

## 1. Pick the flow, not the page

E2E tests cover user journeys from the acceptance criteria: "user submits a
refund and sees it in the list" — not "the refund page renders". If unit
tests already cover it, don't duplicate at E2E cost.

## 2. Data setup

[Adapt to repo strategy: seeded test account / API-created fixtures in
beforeEach / MSW-mocked backend.] Rules regardless of strategy:
- Each test creates or owns its data — never depends on data another test made
- Clean up or use isolated accounts; parallel runs must not collide

## 3. The spec

```ts
test('user submits a refund and sees it listed', async ({ page }) => {
  await page.goto('/orders/123')
  await page.getByTestId('orders-row-menu').click()
  await page.getByRole('menuitem', { name: 'Refund' }).click()
  await page.getByTestId('refund-amount-input').fill('25.00')
  await page.getByTestId('refund-submit').click()
  await expect(page.getByRole('alert')).toContainText('Refund submitted')
  await expect(page.getByTestId('refunds-table')).toContainText('$25.00')
})
```

- Selectors: role/label first, `getByTestId` when ambiguous (see add-testids)
- **Web-first assertions only** (`await expect(...)`) — never
  `waitForTimeout`; a sleep in a spec is a bug
- Assert the outcome the USER sees, not network responses

## 4. Stability pass

Run it 5x locally (`--repeat-each=5`). Flaky now = flaky forever in CI —
fix before merging (see fix-flaky-test).

## 5. Wire into CI

[Adapt: tag/project conventions, which suite it joins.] Note the new spec in
the PR description.
