---
name: new-component
description: Create a React component the way this app expects — folder layout, props/tokens rules, CSS module, tests, and no data fetching inside components.
---

# New component

## 1. Decide where it lives

- Reused across pages → `src/components/<component-name>/`
- Only used by one page → `src/pages/<page>/components/`

## 2. Create the folder (no generator in this repo — follow this layout exactly)

```
src/components/date-range-picker/
  date-range-picker.tsx
  date-range-picker.spec.tsx
  date-range-picker.module.css   # only if it has styles
  index.ts                       # re-export
```

## 3. Component shape

```tsx
// date-range-picker.tsx
import styles from './date-range-picker.module.css'

export interface DateRangePickerProps {
  value: { from: Date; to: Date }
  onChange: (range: { from: Date; to: Date }) => void
  disabled?: boolean
}

export function DateRangePicker({ value, onChange, disabled }: DateRangePickerProps) {
  // ...
}
```

Rules:
- Props: `interface <Component>Props`, exported.
- Styles: CSS module using variables from `src/styles/tokens.css` — no raw hex
  colors or px font sizes.
- **No data fetching inside components.** Server data comes from a React Query
  hook in `src/api/`, called by the page and passed down as props (or the hook
  itself passed for lists). If the hook doesn't exist yet, create it in
  `src/api/` first.

## 4. Test

`date-range-picker.spec.tsx` — Testing Library, query by role/label. Minimum:
renders with required props, fires `onChange` on interaction, respects `disabled`.

## 5. Verify

```
pnpm typecheck && pnpm lint && pnpm test
```
