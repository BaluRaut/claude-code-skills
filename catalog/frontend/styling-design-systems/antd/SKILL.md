---
name: antd
description: Use Ant Design correctly at this repo's pinned version — theme via ConfigProvider tokens (never .ant-* CSS overrides), forms, tables, i18n/locale, icons, and upgrade-safe patterns.
---

# Ant Design

Pinned version: **[5.x.y — adapt; run `pnpm why antd`]**. Version matters
enormously: v4 themes with less-variable overrides, v5 themes with the
CSS-in-JS token system — advice for one is wrong for the other. When unsure
about an API, check the docs FOR THIS VERSION, not latest.

## 1. Styling antd — the one rule

**Theme through ConfigProvider tokens; never write CSS against `.ant-*`
classes.** Class-name overrides break silently on every antd upgrade.

Escalation order when a component doesn't look right:
1. Global/design token in ConfigProvider (see theming skill — tokens map
   from `tokens.ts`)
2. Component-level token: `theme={{ components: { Table: { headerBg: … } } }}`
3. The component's `styles`/`classNames` props (v5)
4. Last resort: `styled(Component)` wrapper with `&&` (see styled-components
   skill) — and a comment saying why tokens couldn't do it

## 2. Forms

[Adapt to repo reality — pick one and delete the other:]
- **antd Form**: `Form.useForm()`, rules on `Form.Item`, validation messages
  as i18n keys via `validateMessages`; controlled by the form instance — don't
  mix in your own useState per field
- **react-hook-form + antd inputs**: antd inputs wrapped in `Controller`;
  validation lives in zod (see new-form) — antd `rules` stay empty

## 3. Tables (the antd workhorse)

- `columns` defined OUTSIDE render (or memoized) — inline columns re-create
  every render and kill sort/filter state
- Server-driven data: controlled `pagination`/`sorter`/`filters` via
  `onChange`, wired to the query hook's params (see new-data-hook)
- Always set `rowKey` to the entity id — index keys break selection
- Row testids via `onRow` → `data-testid="orders-row"` + entity id attribute

## 4. i18n, dates, icons

- `ConfigProvider locale={...}` synced with the app locale [adapt: antd locale
  import + dayjs locale must switch together]
- antd v5 uses **dayjs** — don't import moment; date pickers take dayjs objects
- Icons from `@ant-design/icons`, named imports only

## 5. DoD on antd components

- testids: antd renders wrapper DOM — verify the testid lands on the
  interactive element (`data-testid` prop passes through on most; check
  Select/DatePicker which need [`data-testid` via props on the wrapper — verify per component])
- All texts (placeholders, empty states `locale` props, confirm buttons)
  through i18n
- Modals/Popconfirm: focus behavior verified keyboard-only (a11y-audit)

## 6. Upgrades

antd minor bumps are usually safe; ANY antd upgrade = one dedicated PR
(upgrade-deps skill) + visual pass over Table/Form/Modal/Select screens —
those break most.
