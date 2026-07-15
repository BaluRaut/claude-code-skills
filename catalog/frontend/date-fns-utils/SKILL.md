---
name: date-fns-utils
description: date-fns conventions — central date helpers, parseISO not new Date, i18n-locale formatting, timezone rules, and coexistence with antd's dayjs.
---

# date-fns

Version: [v3/v4 — adapt; v3 went pure-ESM named exports]. 

**The coexistence decision first** [adapt per repo]: antd v5 ships dayjs, so
antd repos already have TWO date libraries. House rule: date-fns for app
logic/formatting; dayjs objects ONLY at the antd component boundary
(DatePicker values), converted immediately (`date.toDate()` /
`dayjs(jsDate)`). If a repo prefers to standardize on dayjs alone, that's
valid — then this skill's patterns apply with dayjs APIs. Never a third
library, and moment is banned in new code.

## 1. Central helpers — components don't format dates

All formatting/parsing through [src/lib/dates.ts — adapt]:

```ts
import { parseISO, format } from 'date-fns'

export const parseApiDate = (iso: string) => parseISO(iso)
export const formatDate = (d: Date) => format(d, 'PP', { locale: currentLocale() })
export const formatDateTime = (d: Date) => format(d, 'PPp', { locale: currentLocale() })
```

One place defines the app's date formats; a component importing `format`
directly is a review catch — formats drift otherwise.

## 2. Parsing rules

- API dates are ISO strings → `parseISO`, NEVER `new Date(string)`
  (implementation-dependent parsing — the classic Safari bug class)
- date-fns is immutable — every function returns a new Date; code "mutating"
  a date is broken code
- Comparisons: `isBefore`/`isAfter`/`differenceInDays` — never `<` on dates
  or manual `getTime()` math for calendar concepts (DST breaks day math)

## 3. Locale & i18n

- Locale objects loaded per app locale [dynamic import map — adapt] and fed
  through the central helpers — the raw `format` default is English-only
- Localized format tokens (`PP`, `PPp`) over hand-built patterns
  (`dd/MM/yyyy` hardcodes one region's order); relative time
  (`formatDistanceToNow`) also takes the locale

## 4. Timezones

- Store/transport UTC ISO; display in the user's zone — the boundary is the
  display layer, nowhere else
- Zone-aware operations ("9am in the store's timezone"): `@date-fns/tz` /
  `date-fns-tz` [adapt] — never manual offset arithmetic
- Tests: pin `TZ` in the runner config [adapt] or date tests flake between
  laptops and CI (fix-flaky-test cause #4)
