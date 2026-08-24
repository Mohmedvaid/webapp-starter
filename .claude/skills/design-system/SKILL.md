---
name: design-system
description: "Styling, theming, and component appearance. Use when adding or changing a component's look, picking colors or spacing, adding a variant, implementing dark mode, checking contrast or accessibility, or when about to write any literal color, size, font, or radius. Triggers include 'style this', 'what color', 'spacing', 'theme', 'dark mode', 'this looks off', 'add a variant', 'contrast', 'accessible'. Enforces the token contract, the five-state matrix, and the accessibility baseline from docs/design-system.md. Do NOT use for component logic, data fetching, or the server and client boundary; that is frontend-patterns."
---

# Design system

One rule carries the rest: **no literal color, size, font, radius, or duration appears in a component.** Lint enforces it. If lint passes and a literal exists, the rule has a hole worth fixing.

## The contract

```
component  ->  tailwind utility  ->  CSS variable  ->  value
```

Changing a brand color is a one-line edit in `src/styles/tokens.css`. If it is not, the chain is broken somewhere.

Token names are semantic, never literal. `--color-danger`, not `--color-red`. The day danger becomes orange, a literal name becomes a lie that spreads.

## Before adding a component

Ask in order:

1. Does it exist in the inventory in `design-system.md`?
2. Can it be composed from existing components?
3. Is it a variant of an existing component rather than a new one?

Only after all three are no does a new component get built. It then gets added to the inventory in the same commit, or the inventory is already fiction.

## Variants

Defined with `cva` or equivalent, in one place, with a typed API. Never prop-driven string concatenation, which produces classes Tailwind cannot see and silently drops from the build.

A variant that is used once is not a variant. It is a one-off, and it belongs inline until a second use appears.

## The five states

Every data-driven surface handles all five. The empty state is the one that gets skipped and the one users hit first.

| State | Requirement |
|---|---|
| Loading | Skeleton matching the real layout, not a spinner. Prevents layout shift |
| Empty | Says what would be here and gives the action that creates the first one |
| Error | What failed, whether retrying helps, and the retry control. Never a raw error string |
| Partial | Some loaded, some failed. Show what worked, mark what did not |
| Success | The content |

Every interactive element handles default, hover, focus-visible, active, disabled, and loading. Focus-visible is never removed, only restyled.

## Dark mode

Token values swap at the root. No component contains a dark-mode conditional. If a component needs one, the token set is missing a semantic role, so add the role rather than the conditional.

## Accessibility baseline

Not aspirational. Checked by axe in CI on every shipped page.

- Contrast 4.5:1 body text, 3:1 large text and UI boundaries
- Everything interactive is keyboard reachable and operable, in a sensible order
- Focus always visible, skip link before the navigation
- Form errors associated with their field and announced
- Alt text present, or empty when decorative. Never a filename
- Headings descend without skipping, one `h1` per page
- Touch targets 44px minimum
- Nothing conveys meaning by color alone

## Motion

Three durations, two curves, all respecting reduced-motion preference. Anything over roughly 320ms reads as broken rather than smooth.

## Review checks

- Any literal value in a component
- A new component that duplicates an existing one
- A missing empty or error state
- A removed focus ring
- A dark-mode conditional inside a component
- Meaning carried by color with no text or icon backup
