---
name: code-design-solid
description: Design standards for React code — SOLID applied honestly, size and cognitive-load limits, logic extracted into pure testable functions, DRY with the rule of three.
---

# Code design (frontend)

The measurable rules first — they catch 80% of design problems without any
philosophy.

## 1. Size & cognitive-load limits [numbers — adapt per repo]

- Component file: **~250 lines** → split (refactor-component has the
  patterns); component function ~100 lines
- Plain function: ~40 lines, nesting ≤ 3 levels (early returns flatten),
  params ≤ 3 (then an options object)
- Booleans multiplying (`isEditing && !isLocked && (isAdmin || isOwner)`)
  → name the concept: `const canEdit = ...` or a discriminated state
  (typescript §4)
- Limits are review TRIGGERS, not laws — crossing one means "justify or
  split", not auto-reject

## 2. The testability rule (the one that changes behavior most)

**If it has logic, it's a pure function — extractable and tested without
rendering anything.** Price math, filtering/sorting, validation beyond the
schema, permission predicates: live in plain files (`src/lib/`, feature
`utils.ts`), take data in, return data out. Components render; hooks
orchestrate; functions decide. If testing a rule requires mounting a
component, the rule is in the wrong place (write-unit-tests gets 10x
cheaper this way).

## 3. SOLID, translated to React (the honest version)

- **S** — one reason to change: a component that fetches + transforms +
  renders + tracks = hook (fetch) + pure fn (transform) + component
  (render) + catalog call (track)
- **O** — extend via composition: `children`/slots/render props over a new
  boolean prop per variant; 5+ boolean props = the component wants to be
  composed, not configured
- **L** — a specialized component (`PrimaryButton`) accepts everywhere its
  base works; don't break the base's contract (drop `onClick`, ignore
  `disabled`)
- **I** — narrow props: `{ customerId, customerName }` or a scoped
  `CustomerSummary`, not the whole `Customer` — fat props create phantom
  coupling and re-renders
- **D** — components depend on seams, not concretions: hooks from
  data-access (not axios), `can()` (not `user.role ===`), the analytics
  catalog (not the SDK). The seams are what make tests and vendor swaps
  cheap

## 4. DRY — with the rule of three

- Abstract on the THIRD occurrence, when the shape is stable — the wrong
  abstraction costs more than duplication (a shared component serving two
  masters grows the boolean-prop disease of §3-O)
- DRY is about KNOWLEDGE, not lines: two identical-looking blocks that
  change for different reasons should stay separate; one business rule in
  two places must be unified even if the code looks different
- Where DRY always pays: types from schemas (typescript §5), tokens
  (theming), route builders (react-router-v6), query keys (new-data-hook)
- No `utils.ts` dumping ground — shared code gets a named module
  (`lib/money.ts`), or it stays local until it earns one
