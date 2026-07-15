---
name: component-review
description: Audit a component or screen against the house thresholds — size, complexity, props/state/effect counts, inline-logic testability, duplication. Reports findings with the fixing skill; does not rewrite unless asked.
---

# Component review

Review skill: run on demand ("review OrderTable") or as a PR pass. Output
is a severity-ordered findings list — `file:line — rule — suggested fix
(which skill)`. Do NOT rewrite code unless explicitly asked to fix.

## 1. Size & structure [thresholds — adapt, from code-design-solid]

- File > ~250 lines / component fn > ~100 / any fn > ~40 → split
  (refactor-component)
- JSX nesting > 3 levels → extract subcomponents; conditional pyramids →
  early returns or a state enum (typescript §4)
- Render helpers returning JSX inside the component body → real child
  components (they break memoization and readability)

## 2. Complexity signals (count them, report the numbers)

- **Props > ~8**, or ≥5 booleans → wants composition, not configuration
  (code-design-solid §3-O); whole-entity props where 2 fields are used
- **useState > ~5** interacting flags → `useReducer` / discriminated state
- **useEffect > ~3** → most are probably derived values or handlers in
  disguise — run each through react-effects §1's decision tree; flag any
  effect without cleanup that starts something
- Inline objects/arrays/functions passed to memoized children → stability
  bug (react-effects §4)
- `useMemo`/`useCallback`/`React.memo` with no measured hot path → noise;
  with an unstable dep list → broken (perf-audit)

## 3. Testability (code-design-solid §2 applied as an audit)

- Multi-line logic inside JSX handlers
  (`onClick={() => { ...10 lines... }}`) → named function; if it DECIDES
  something → pure function in lib/, tested without rendering
- Direct imports of axios/SDKs/storage in the component → the seam is
  missing (http-client, analytics catalog, storage wrapper)
- Hidden state: module-level variables, singletons mutated from render,
  reading globals mid-render → inject or lift

## 4. Duplication (DRY audit, rule-of-three)

- Same JSX block ≥3× → extract component; same fetch/transform/validation
  logic vs existing hooks/utils → point at the EXISTING one
  (new-data-hook keys, zod schemas) — the finding names both locations

## 5. DoD spot-check (cheap while you're here)

Hardcoded user-facing strings (add-translation), interactive elements
missing testids (add-testids), missing loading/error/empty states
(new-page §3), swallowed catches (error-monitoring).

## 6. Report format

Max ~10 findings, worst first, each: location, rule, one-line why, the
skill that fixes it. End with the two-sentence overall verdict ("healthy /
needs a split / redesign candidate"). Counts you measured (lines, props,
effects) go in the finding — numbers beat adjectives.
