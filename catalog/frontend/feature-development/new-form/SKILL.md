---
name: new-form
description: Build a form the house way — schema-first validation, design-system fields, submit states, server-error mapping, accessibility, tests.
---

# New form

Stack: [react-hook-form + zod — adapt to repo].

## 1. Schema first

```ts
const refundFormSchema = z.object({
  orderId: z.string().uuid({ message: 'validation.order_id' }), // i18n keys, not English
  amount: z.coerce.number().positive({ message: 'validation.amount_positive' }),
  reason: z.enum(['damaged', 'wrong_item', 'other']),
})
type RefundFormValues = z.infer<typeof refundFormSchema>
```

Validation messages are **i18n keys**, translated at render.

## 2. The form

- Fields come from the design system ([libs/ui / src/components — adapt]) —
  never raw `<input>`.
- `useForm({ resolver: zodResolver(refundFormSchema), defaultValues })` —
  defaultValues always, or controlled/uncontrolled warnings follow.
- Inline errors under each field; on submit, focus the first invalid field.

## 3. Submit states (all four)

1. Submitting: button disabled + loading state — double-submit is impossible
2. Server validation errors: map field errors via `setError`; non-field errors
   in a form-level alert
3. Success: [toast + navigate / inline confirmation — adapt to repo pattern]
4. Unexpected failure: form-level alert with retry; error reported to monitoring

## 4. Accessibility

- Every field has a visible `<label>` (design-system field handles it — verify)
- Errors linked via `aria-describedby`; form-level alert has `role="alert"`
- The whole form works keyboard-only — tab through it once yourself

## 5. DoD pass

i18n on all strings including errors; testids on fields + submit; analytics
event on successful submit (not on click).

## 6. Tests

Validation error shows for bad input · valid submit calls the mutation with
correct payload · server field-error renders on the right field · submit
button disables while pending.

## Verification contract

The canonical example of the contract every skill states (see
[../../../AUTHORING.md](../../../AUTHORING.md)).

- **Inputs**: the API contract (endpoint + request/response schema), the
  field-level validation rules, the UI/design for the form.
- **Outputs**: a schema-first form component, a typed submit handler wired to
  the mutation hook, field- and form-level error rendering, tests.
- **Verify**: `typecheck` · `test` (the four cases in §6) · run the form in
  the app and submit once valid, once invalid.
- **Failure modes** (what review + tests must catch): hardcoded validation
  messages instead of i18n keys; double-submit (button not disabled while
  pending); server field-errors dropped instead of mapped via `setError`;
  analytics fired on click instead of on success; missing `defaultValues`
  (controlled/uncontrolled warning); inputs without labels (a11y).
