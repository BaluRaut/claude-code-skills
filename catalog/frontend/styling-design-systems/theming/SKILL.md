---
name: theming
description: Work with the app theme — design tokens as single source of truth, feeding both styled-components and antd ConfigProvider, dark/light mode, adding new tokens.
---

# Theming

One source of truth: the token file [src/theme/tokens.ts — adapt]. Both
consumers derive from it — never define a color twice:

```
tokens.ts ──▶ styled-components ThemeProvider (custom components)
         └──▶ antd ConfigProvider theme      (antd components)
```

## 1. Token rules

- Components use **semantic** tokens (`colors.surface`, `colors.textMuted`,
  `colors.danger`), never raw palette entries (`blue500`) and never hex
  literals. Raw palette → semantic mapping happens once, in the theme file.
- Spacing/radius/typography come from tokens too — no magic `px` in components.
- A new color needs BOTH modes defined (light + dark) and a contrast check
  (4.5:1 text, 3:1 UI) before it ships.

## 2. Wiring (both providers, same tokens)

```tsx
// [src/theme/ — adapt]
<ConfigProvider theme={{
  algorithm: mode === 'dark' ? theme.darkAlgorithm : theme.defaultAlgorithm,
  token: { colorPrimary: tokens.colors.primary, borderRadius: tokens.radius.md, /* map, don't restate */ },
}}>
  <ThemeProvider theme={buildTheme(mode)}>{children}</ThemeProvider>
</ConfigProvider>
```

Antd tokens are MAPPED from your tokens — if a value appears in the
ConfigProvider that isn't derived from `tokens.ts`, that's a bug.

## 3. Dark mode

- Mode state lives in [theme context — adapt]: user preference persisted,
  default from `prefers-color-scheme`
- Components never check the mode (`mode === 'dark' ? ... : ...` in a
  component is a smell) — semantic tokens already resolve per mode
- Images/illustrations/shadows that don't work in both modes get token-driven
  variants, not inline conditionals

## 4. Adding/changing a token

1. Add to `tokens.ts` — both modes, semantic name, comment on intended use
2. Map into the antd ConfigProvider if antd components should reflect it
3. Type it (the styled-components `DefaultTheme` declaration picks it up)
4. Verify BOTH modes visually on the screens that use it — toggle in dev

## 5. Verify

Toggle light/dark across the changed screens; grep the diff for hex literals
and raw px — anything found goes through tokens instead.
