# Phase 1: Design system and shell

**Skills:** `design-system`, `frontend-patterns`
**Gate:** `docs/phases.md`, Phase 1

Tokens exist, nothing bypasses them, and the app has a shape.

## Build

**`src/styles/tokens.css`.** The full contract from `docs/design-system.md`. Semantic names only. Dark values under `:root[data-theme="dark"]`.

**Tailwind config.** Reads the CSS variables. No literal values in the config either.

**`src/components/ui/`.** Button, Input, Textarea, Select, Checkbox, Radio, FormField, Dialog, Sheet, Toast, Table, Card, Badge, Skeleton, EmptyState, ErrorBoundary. Variants via `cva`. Every interactive component handles default, hover, focus-visible, active, disabled, loading.

**App shell.** Root layout, navigation, responsive breakpoints, skip link before the nav, theme toggle.

**Route-level states.** `loading.tsx`, `error.tsx`, `not-found.tsx`. Error boundaries per route, never one at the root only.

**axe.** Wired into the Playwright setup so accessibility runs on every e2e route.

## Avoid

- Any literal color, size, font, radius, or duration inside a component. If lint passes and one exists, fix the rule.
- Prop-driven class string concatenation. Tailwind cannot see those classes and drops them from the build.
- A dark-mode conditional inside a component. If one is needed, the token set is missing a semantic role. Add the role.
- Removing the focus ring. Restyle it, never remove it.
- Building a component the inventory does not list, without adding it to the inventory in the same commit.
- A spinner where a skeleton belongs. Skeletons prevent layout shift, spinners cause it.

## Gate

```
pnpm gate
pnpm test:e2e -- a11y
```

Then: grep `src/components` and `src/features` for hex codes and raw px values. Zero results. Change one token value and confirm the whole app changes. Tab through a page from first element to last.
