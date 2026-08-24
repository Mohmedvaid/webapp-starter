# Phase 4: Payments

**Skills:** `payments-stripe`, `tdd-workflow`, `api-and-validation`
**Gate:** `docs/phases.md`, Phase 4

Read `.claude/skills/payments-stripe/references/event-map.md` before writing a handler and `test-matrix.md` before claiming it is tested.

Build both one-time and subscription paths. `project-init` deletes the unused one per clone.

## Build

**Session creation.** `src/features/billing/server/checkout.ts`. Writes the local `orders` row before redirecting. Idempotency key derived from your own intent, not a random value.

**Webhook route.** `src/app/api/webhooks/stripe/route.ts`. Raw body, signature verified, event id inserted into `stripe_events` with a unique constraint, fast 2xx, state writes synchronous, slow work deferred.

**Fulfillment.** `src/server/fulfillment.ts`. A function of state, not of the event. No-op if already fulfilled. Transactional. Unique constraint on the entitlement enforces it, not an if.

**Handlers.** The eight events in the event map. Both one-time and subscription.

**Entitlement.** `src/server/entitlements.ts` plus `src/core/access.ts`. Access is a pure function of entitlement and current time. Three separate concepts: payment, entitlement, access.

**Reconciliation.** `scripts/reconcile.ts` with `--dry-run`. Lists provider sessions from the last few days with a paid status, asserts a matching fulfilled order, fulfills and alerts on drift.

**Delayed payment methods.** Handled via the async events, or disabled explicitly at session creation. Not silently assumed instant.

## Avoid

- Fulfilling on the success URL. The tab can close before the redirect.
- Treating `checkout.session.completed` as proof money moved. Gate on paid status or the async success event.
- A body parser in front of the signature check.
- Dedupe with an `if` instead of a unique constraint. The if loses the race.
- Swallowing a handler error. Return non-2xx so the provider retries.
- Skipping the reconciliation job because webhooks "work". One will be lost.
- Wrapping the provider SDK in an abstraction so you could "switch providers". Nobody switches, and the wrapper would encode this provider's model anyway. Keep the calls in one module instead.

## Gate

```
pnpm gate
pnpm test:e2e -- checkout
stripe trigger checkout.session.completed
stripe trigger checkout.session.completed
pnpm reconcile --dry-run
```

The second trigger must be a no-op with no duplicate entitlement. Then the manual check: pay, then close the tab immediately, before any redirect. The entitlement still lands.
