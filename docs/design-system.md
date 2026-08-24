# Design System

> Pre-filled with the token contract, which does not change per project. Values change per project.
> The rule that makes this real: no literal color, size, font, radius, or duration appears in a component. Enforced by lint, not by review.

## Token contract

Tokens are CSS custom properties in `src/styles/tokens.css`. Tailwind reads them. Components read Tailwind. Nothing reads a raw value.

```
component  →  tailwind utility  →  CSS variable  →  value
```

Changing a brand color is a one-line edit in one file. If it is not, the contract has been broken somewhere and the lint rule has a hole.

### Color

Semantic names, not literal ones. `--color-danger`, never `--color-red`. The day danger becomes orange, a literal name becomes a lie.

| Token | Role |
|---|---|
| `--color-bg` / `--color-bg-subtle` / `--color-bg-elevated` | Surfaces, back to front |
| `--color-fg` / `--color-fg-muted` / `--color-fg-subtle` | Text, decreasing emphasis |
| `--color-border` / `--color-border-strong` | Dividers and outlines |
| `--color-primary` / `--color-primary-fg` | Primary action and its text |
| `--color-danger` / `--color-danger-fg` | Destructive action and its text |
| `--color-success` / `--color-warning` / `--color-info` | Status only, never decoration |
| `--color-focus` | Focus ring. One value, used everywhere, never removed |

Dark mode swaps values at `:root[data-theme="dark"]`. No component contains a dark-mode conditional.

<!-- FILL: actual hex values per role, light and dark -->

### Type

| Token | Use |
|---|---|
| `--font-sans` / `--font-mono` | Two families maximum. A third needs a reason |
| `--text-xs` through `--text-4xl` | A fixed scale. No arbitrary sizes |
| `--leading-tight` / `--leading-normal` / `--leading-relaxed` | Three, not a continuum |
| `--weight-normal` / `--weight-medium` / `--weight-bold` | Three, not nine |

<!-- FILL: chosen families and scale -->

### Space, radius, motion

| Token | Notes |
|---|---|
| `--space-1` through `--space-16` | One scale, geometric. Every margin and padding comes from it |
| `--radius-sm` / `--radius-md` / `--radius-lg` / `--radius-full` | Four values |
| `--duration-fast` (120ms) / `--duration-base` (200ms) / `--duration-slow` (320ms) | Anything slower feels broken |
| `--ease-out` / `--ease-in-out` | Two curves |

All motion respects `prefers-reduced-motion`. Not optional, not a nice-to-have.

## State matrix

Every data-driven surface handles five states. The empty state is the one that gets skipped, and it is the one users hit first.

| State | Requirement |
|---|---|
| Loading | Skeleton matching the real layout, not a spinner. Prevents layout shift |
| Empty | Explains what would be here and gives the action that creates the first one |
| Error | Says what failed, whether it is retryable, and gives the retry. Never a raw error string |
| Partial | Some data loaded, some failed. Show what worked, mark what did not |
| Success | The actual content |

Every interactive element handles: default, hover, focus-visible, active, disabled, loading. Focus-visible is never removed, only restyled.

## Component inventory

| Component | Status | Variants | Notes |
|---|---|---|---|
| Button | | primary, secondary, ghost, danger | Loading state built in, disabled during pending action |
| Input | | text, email, password, number | Label always present. Placeholder is never the label |
| Select | | | Keyboard navigable, native on mobile |
| Checkbox / Radio | | | Label is the click target |
| Textarea | | | Character count when limited |
| Form field wrapper | | | Owns label, description, error, and their aria wiring |
| Dialog | | | Focus trap, escape closes, focus returns to trigger |
| Sheet | | | Mobile drawer equivalent |
| Toast | | success, error, info | Never the only notice of a failure |
| Table | | | Sortable header, empty state, mobile card fallback |
| Card | | | |
| Badge | | status colors | Text, not color alone, carries the meaning |
| Skeleton | | | Matches the shape it replaces |
| Empty state | | | Illustration optional, action required |
| Error boundary | | | Per-route, not one at the root |

<!-- FILL: mark status as each is built in phase 1 -->

## Accessibility baseline

Non-negotiable, checked by axe in CI on every shipped page.

- Contrast 4.5:1 for body text, 3:1 for large text and UI boundaries.
- Every interactive element is reachable and operable by keyboard, in a sensible order.
- Focus is always visible. A skip link precedes the main navigation.
- Form errors are associated with their field via `aria-describedby` and announced.
- Images have alt text, or `alt=""` when decorative. Never a filename.
- Headings descend without skipping. One `h1` per page.
- Touch targets 44px minimum.
- Nothing conveys meaning by color alone.

## Voice

<!-- FILL: how the product talks. Error messages especially. Terse and plain, or warm and explanatory. Pick one and hold it. -->

| Surface | Rule |
|---|---|
| Buttons | Verb first, what happens on click |
| Errors | What went wrong, what to do about it. Never "an error occurred" |
| Empty states | What goes here, and how to add the first one |
| Confirmations | Name the consequence, especially when irreversible |
