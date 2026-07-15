---
name: testing-library-react
description: React Testing Library conventions — query priority, user-event over fireEvent, async patterns, the shared custom render with providers.
---

# React Testing Library

The component-testing layer under both runners (see vitest-unit/jest-unit);
what to test comes from write-unit-tests — this is HOW.

## 1. Query priority (in order — reviewer checks this)

1. `getByRole('button', { name: /submit refund/i })` — role + accessible name
2. `getByLabelText` for form fields
3. `getByText` for non-interactive content
4. `getByTestId` — last resort, when no accessible handle exists (and that
   itself may be an a11y bug — see a11y-audit)

Banned: container.querySelector, class/DOM-structure queries. If a test
can't find an element by role/label, users of assistive tech can't either.

## 2. get / query / find — not interchangeable

- `getBy*` — element must be there NOW (throws otherwise)
- `queryBy*` — ONLY for asserting absence: `expect(queryByRole(...)).not.toBeInTheDocument()`
- `findBy*` — anything that appears after async work; this replaces most
  `waitFor` wrapping

`waitFor` rules when you do need it: assertions only inside — never actions
(clicks inside waitFor re-fire on every retry).

## 3. user-event, not fireEvent

```ts
const user = userEvent.setup()
await user.type(screen.getByLabelText(/amount/i), '25')
await user.click(screen.getByRole('button', { name: /submit/i }))
```

user-event simulates real interaction (focus, keydown, pointer events);
fireEvent bypasses it and hides bugs. All user-event calls are awaited.

## 4. The shared custom render

One `renderWithProviders` in [src/test-utils.tsx — adapt] wrapping theme,
fresh QueryClient (retry: false), router (MemoryRouter with initial route),
i18n test instance. Tests import render from test-utils, NEVER from
@testing-library/react directly — [enforce via lint rule — adapt].

## 5. What a good component test looks like

Arrange props/handlers → interact via user-event → assert what the USER
sees (rendered output, accessible states) and the contract calls (onChange,
mutation, analytics event). Never assert hooks' internal state, render
counts, or styled-components class names.
