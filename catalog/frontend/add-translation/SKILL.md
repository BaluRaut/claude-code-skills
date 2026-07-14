---
name: add-translation
description: Add or change user-facing text through i18n — key naming, plurals and interpolation, locale TODO markers, and the string-concatenation trap.
---

# Add a translation

i18n stack: [react-i18next / lingui — adapt]. Rule zero: **no user-visible
string literal in a component** — including aria-labels, placeholders, alt
text, error toasts.

## 1. Key

- Naming: `<feature>.<screen>.<element>` → `refunds.form.submit_button`
- Never reuse a key for different meaning because the English text matches —
  "Close" (button) and "Close" (status) translate differently in other languages
- Changed meaning = new key (translators can't detect changed context)

## 2. Add to locales

- `en.json`: the real string, in the feature's namespace
- Other locales: [English fallback + TODO marker / translation-queue process —
  adapt]. Never machine-translate silently into locale files.

## 3. Plurals & interpolation — the traps

```ts
t('refunds.list.count', { count })       // "{{count}} refund" / "{{count}} refunds" — use the plural form, never `count + ' refunds'`
t('refunds.form.min_amount', { amount }) // interpolate — NEVER concatenate: t('a') + amount + t('b') breaks word order in most languages
```

Dates/numbers/currency: format with [Intl helpers — adapt], not string math.

## 4. Verify

- Run with [missing-key warnings enabled — adapt]: console clean
- Switch to the longest locale ([de/fr]) and check the layout doesn't break
- Grep your diff for quoted strings in JSX — anything user-visible left?
