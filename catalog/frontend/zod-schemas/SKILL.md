---
name: zod-schemas
description: Zod conventions — schemas as the single source of types, parsing at boundaries (API/env/forms), i18n-ready messages, composition patterns.
---

# Zod schemas

Version: [v3 / v4 — adapt; v4 changed error customization APIs — check
`pnpm why zod`].

## 1. Schema = the type (single source of truth)

```ts
export const refundSchema = z.object({
  id: z.string().uuid(),
  amountCents: z.number().int().positive(),
  status: z.enum(['pending', 'approved', 'rejected']),
  createdAt: z.string().datetime(),
})
export type Refund = z.infer<typeof refundSchema>
```

Never hand-write an interface that mirrors a schema — `z.infer` only.
Schemas live [next to the API client / shared package if FE+BE share —
adapt]; forms derive their schemas from entity schemas via
`.pick()`/`.omit()`/`.extend()`, not by redefining fields.

## 2. Parse at boundaries — trust nothing twice

- **API responses**: parse in the client/data layer, so the rest of the app
  works with GUARANTEED shapes. `safeParse` + report to monitoring on
  failure [house choice: fail loud in dev, tolerate+log in prod — adapt]
- **Env/config**: parsed once at startup (`src/lib/env.ts`) — the app
  crashes early with a clear message, not deep in a feature at runtime
- **Forms**: via resolver (see new-form) — `z.coerce.number()` for numeric
  inputs, since form values arrive as strings
- Inside the app (component → component): do NOT re-parse; types carry the
  guarantee

## 3. Messages are i18n keys

`z.string().min(1, { message: 'validation.required' })` — translated at
render, never English literals in schemas (see add-translation). Repo-wide
default messages via [error map — adapt to zod version].

## 4. Composition patterns

- Variants: `z.discriminatedUnion('type', [...])` — not optional-field soup
- Reuse: `.pick`/`.omit`/`.extend` from the entity schema
- Transforms sparingly: parsing should validate shape; heavy mapping belongs
  in named functions after the parse

## 5. Pitfalls

- `z.any()` / blanket `.passthrough()` — deleting the safety you added zod for
- Parsing the same data at multiple layers "to be safe" — boundary only
- Schema drift from the backend: when the API changes, the parse failure in
  monitoring IS the alarm — don't loosen the schema to silence it
