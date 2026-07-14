---
name: styled-components
description: Write styled-components the house way — module-level definitions, transient props, typed theme access, variants, wrapping antd, and the classic pitfalls.
---

# styled-components

Version: [v6.x — adapt; v5→v6 changed transient-prop and types behavior —
check the repo's version before advising on APIs].

## 1. The non-negotiables

- **Define styled components at module top level — NEVER inside a component
  body.** A styled definition inside render gets a new identity every render →
  full remount, lost state, killed performance. This is the #1 review catch.
- Placement per repo convention: [bottom of the component file / separate
  `<component>.styles.ts` — adapt]. Follow what the file already does.
- All values from the theme (see theming skill): `${({ theme }) =>
  theme.colors.surface}` — no hex, no magic px.

## 2. Props → styles

Props that only drive styling are **transient** (`$` prefix) so they don't
leak onto the DOM:

```tsx
const Badge = styled.span<{ $tone: 'success' | 'danger' }>`
  color: ${({ theme, $tone }) => theme.colors[$tone]};
  padding: ${({ theme }) => theme.spacing.xs};
`
```

- Enumerated variants (`$tone`, `$size`) over booleans that multiply
  (`$isBig $isRed $isActive` → unreadable)
- Shared fragments via the `css` helper, exported from [theme/mixins — adapt];
  static HTML attributes via `.attrs()`

## 3. Composition

- Restyle an existing component: `styled(Button)` — requires the base to
  forward `className`
- Global styles ONLY in the app-level `createGlobalStyle` [adapt path] —
  never per-feature globals
- Never target another styled component's internals by class name; nest with
  the component reference (`${Icon} { … }`) if you must

## 4. Wrapping antd

`styled(AntButton)` works but antd's own specificity sometimes wins. Order of
preference: ConfigProvider tokens (theme-level) → component `styles`/`classNames`
prop [v5] → `styled()` wrapper with `&&` specificity boost. Never `!important`
against `.ant-*` classes — it breaks silently on antd upgrades.

## 5. Testing & DoD

Styles are implementation details — test behavior, not CSS strings. Transient
props keep DOM clean for testids; `data-testid` passes through styled wrappers
automatically.
