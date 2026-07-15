---
name: add-testids
description: Add data-testid attributes to a screen for Playwright — naming convention, which elements need them, and treating testids as stable API.
---

# Add testids

Testids are the contract between the app and the Playwright suite. QA writes
tests against them; renaming one is a breaking change.

## 1. Convention

`data-testid="<feature>-<element>[-<action>]"` — kebab-case, stable, meaningful:

```
refund-submit          refund-amount-input       orders-table
orders-row-menu        orders-row-delete         cart-total
```

For repeated rows/items, testid on the row + a data attribute for identity:
`data-testid="orders-row" data-order-id={order.id}` — never index-based ids.

## 2. What gets one

- Every interactive element: buttons, inputs, selects, links-as-actions,
  menu items, toggles
- Every assertion target: headings, totals, status badges, empty/error states,
  toast/alert containers
- NOT every div — testids are for behavior, not layout

## 3. Sweep procedure (for an existing screen)

1. Open the screen; list its user actions and the things a test would assert
2. Add testids per the convention; route them through design-system props
   ([`testId` prop — adapt]) so they land on the right DOM node
3. If a Playwright selectors/page-object file exists [e2e/selectors — adapt],
   register them there

## 4. Verify

Grep the diff: every new interactive element has one. Run the affected
Playwright specs if they exist. Note any RENAMED testid in the PR description
as a breaking change for the E2E suite.
