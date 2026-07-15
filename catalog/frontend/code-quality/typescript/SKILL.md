---
name: typescript
description: TypeScript house rules — strict mode non-negotiable, no any/no !, unknown at boundaries, discriminated unions, inference vs annotation, cast policy.
---

# TypeScript conventions

## 1. Strictness is not optional

`strict: true` (+ `noUncheckedIndexedAccess` [house choice — adapt]) in
every tsconfig. Never loosened per-file to "make it compile" — the error
is telling you about a runtime bug.

## 2. The banned list (review catches)

- **`any`** — use `unknown` and narrow. `any` doesn't silence one error;
  it silences every future error that flows through it. Third-party
  untyped? One typed wrapper at the boundary, `any` contained inside
- **Non-null `!`** — fix the type or handle the case; `user!.id` is a
  runtime crash wearing a type annotation. (Exception: test fixtures
  [house choice — adapt])
- **`as` casts** — a cast is an unchecked claim. Prefer: type guards,
  discriminated-union narrowing, `satisfies` (checks WITHOUT widening).
  A justified cast gets a comment saying why it's safe
- `@ts-ignore`/`@ts-expect-error` without a comment + ticket

## 3. Types at boundaries, inference inside

- Explicit types: exported function signatures, public component props,
  API/event contracts (from zod via `z.infer` — never hand-mirrored)
- Inference: locals, obvious returns — annotating `const x: number = 5`
  is noise
- `unknown` at every trust boundary (API responses, JSON.parse, catch,
  postMessage) → zod-parse or narrow before use (zod-schemas §2)

## 4. Model states so wrong ones can't compile

```ts
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: ApiError }
```

- Discriminated unions over optional-field soup
  (`{ data?, error?, isLoading? }` allows impossible combinations)
- Exhaustive switches: `default: { const _: never = state; }` — adding a
  variant breaks compilation everywhere it's unhandled, which is the point
- String-literal unions over `enum` [house choice — adapt]; `readonly` /
  `as const` for config shapes

## 5. Reuse types, don't re-declare

`Pick`/`Omit`/`Partial` from the source-of-truth type; component prop
types derive from entity types (which derive from schemas). Two
hand-written types describing the same thing WILL drift — that's the DRY
rule applied where it actually pays (code-design-solid §4).
