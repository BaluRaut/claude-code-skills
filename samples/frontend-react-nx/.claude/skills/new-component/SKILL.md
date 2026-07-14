---
name: new-component
description: Create a React component the way this workspace expects — correct project placement, nx generator, design tokens, story, tests, and barrel export.
---

# New component

## 1. Decide where it lives

- Needed by **2+ apps** (or clearly generic UI) → `libs/ui`
- Specific to **one app** → `apps/<app>/src/components/`
- Not sure → start app-local; promoting to `libs/ui` later is cheap.

## 2. Scaffold with the nx generator — do not hand-create files

```bash
# shared design-system component
pnpm nx g @nx/react:component src/lib/date-range-picker/date-range-picker --project=ui

# app-local component
pnpm nx g @nx/react:component src/components/order-summary/order-summary --project=storefront
```

## 3. Flesh it out

```tsx
// libs/ui/src/lib/date-range-picker/date-range-picker.tsx
import { tokens } from '../../tokens'

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
- Colors/spacing/typography come from `tokens` — no raw hex or px literals.
- **No data fetching inside components.** If it needs server data, the hook
  belongs in `libs/data-access` and gets passed in or called by the page.

## 4. Story (required for every `libs/ui` component)

```tsx
// date-range-picker.stories.tsx
import type { Meta, StoryObj } from '@storybook/react'
import { DateRangePicker } from './date-range-picker'

const meta: Meta<typeof DateRangePicker> = { component: DateRangePicker }
export default meta

export const Default: StoryObj<typeof DateRangePicker> = {
  args: { value: { from: new Date('2026-01-01'), to: new Date('2026-01-31') } },
}
export const Disabled: StoryObj<typeof DateRangePicker> = {
  args: { ...Default.args, disabled: true },
}
```

Include one story per meaningful visual state (disabled, error, empty…).

## 5. Test

`date-range-picker.spec.tsx` next to the component. Testing Library, query by
role/label. Minimum: renders with required props, fires `onChange` on
interaction, respects `disabled`.

## 6. Export and verify

1. Add to the barrel: `libs/ui/src/index.ts` → `export * from './lib/date-range-picker/date-range-picker'`
2. Consumers import from the alias: `import { DateRangePicker } from '@acme/ui'`
3. Run: `pnpm nx test ui && pnpm nx lint ui` (swap `ui` for the app project if app-local).
