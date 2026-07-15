---
name: a11y-audit
description: Accessibility pass on a page — automated axe scan, keyboard walk, focus management, labels/roles, contrast. Fix and encode patterns.
---

# Accessibility audit

## 1. Automated scan (catches ~30–40%)

Run [axe DevTools / @axe-core/playwright / vitest-axe — adapt] on the page in
its main states (loaded, error, modal open). Fix everything at
serious/critical; triage the rest.

## 2. Keyboard walk (catches what axe can't)

Unplug your trackpad, tab through the entire page:

- Every interactive element reachable and operable (Enter/Space)
- Visible focus ring everywhere (design-system token, never `outline: none`
  without replacement)
- No keyboard traps; logical tab order matching visual order
- Escape closes modals/menus; focus returns to the trigger afterwards

## 3. Focus management (the SPA-specific bugs)

- Route change: focus moves to the new page's h1 / main landmark [adapt to
  repo's router helper]
- Modal open: focus into modal, trapped inside, restored on close
- Async results (search, filters): announced via live region, or focus moved

## 4. Semantics & labels

- One h1; heading levels don't skip; landmarks (main/nav) present
- Every input labeled (visible label, not placeholder-as-label)
- Icon-only buttons: `aria-label` (translated — see add-translation)
- Images: meaningful alt or `alt=""` if decorative
- Status/error messages: `role="alert"` / live region

## 5. Contrast

Check text and interactive states against tokens (4.5:1 body, 3:1 large/UI).
Failures are TOKEN bugs — fix in the design system, not per-component.

## 6. Close the loop

Fix + regression tests for the worst issues (axe in CI for this page).
Recurring failure patterns → new rule in CLAUDE.md so AI-written code stops
producing them.
