# Phase 6: Public surface

**Skills:** `frontend-patterns`, `design-system`
**Gate:** `docs/phases.md`, Phase 6

## Build

**Metadata.** Per-route title and description, Open Graph, Twitter card, canonical URLs.

**`sitemap.ts` and `robots.ts`.** Both gated by `INDEXING_ENABLED`. **Defaults to noindex when the variable is unset.** Defaulting the other way means an unset variable publishes a half-built site to crawlers, and crawler caches are slow to undo.

**Marketing pages.** Home, pricing, and whatever the charter implies. Pricing matches `docs/product.md` exactly. A pricing page that disagrees with the checkout is a support ticket generator.

**Legal.** Terms, privacy, refund policy. Placeholder content with a clear marker, since the real text is Mohmed's decision, not a generated one.

**Support contact.** Reaches a real inbox. Verify it actually delivers.

**Analytics.** No personal data in event payloads. Event names as constants, not inline strings.

**Lighthouse.** Wired into `pnpm gate`, run against a production build rather than the dev server. Asserts the budgets declared in `docs/product.md` and **fails the gate** when one is breached. A budget that only produces a number is a report, not a gate.

## Avoid

- Generating legal text and presenting it as ready. Mark it as placeholder and say so plainly.
- Indexing defaulting to enabled.
- Marketing copy duplicating the pricing numbers in a second place. One source, referenced.
- Client components for static marketing pages.
- Analytics that fire before consent where consent is required.
- Running Lighthouse against the dev server. Those numbers mean nothing.
- Recording a performance number without a threshold that fails.

## Gate

```
pnpm gate
curl localhost:3000/robots.txt
```

Adversarial check: unset the indexing variable entirely and confirm robots still disallows. Then breach a budget deliberately and confirm the gate fails rather than reporting.
