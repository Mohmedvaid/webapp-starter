---
name: payments-stripe
description: "Build or review anything touching payments. Use for checkout sessions, webhooks, fulfillment, entitlements, refunds, disputes, subscriptions, billing portal, reconciliation, or price changes. Triggers include 'checkout', 'stripe', 'webhook', 'payment', 'subscription', 'refund', 'chargeback', 'they paid but did not get access', 'billing'. Load references/event-map.md before writing a handler and references/test-matrix.md before claiming payments are tested. Enforces webhook-as-source-of-truth, idempotent fulfillment, and the reconciliation job. Do NOT use for pricing strategy or what to charge; that is a business decision, not an implementation one."
---

# Payments

The failure mode is not a crash. It is a customer who paid and got nothing, discovered days later.

Load `references/event-map.md` before writing a handler. Load `references/test-matrix.md` before claiming payments are tested.

## Invariants

1. **Never fulfill on the success URL.** The customer can close the tab between paying and redirecting. That URL is a courtesy, not a signal.
2. **The webhook is the source of truth.** Client state is a hint.
3. **Every handler is idempotent.** The same event can arrive twice. Dedupe on the event id with a unique constraint, not an `if` statement.
4. **Verify the signature against the raw body.** Any parser in front of the route corrupts the check.
5. **Return 2xx fast, write state synchronously, defer the slow parts.** Emails and PDFs go after the state write, never in front of it.
6. **`checkout.session.completed` does not mean money moved.** Gate on paid status or the async success event.
7. **Idempotency key on every write call to the provider**, keyed by your own intent, so a retry never double-charges.
8. **Payment, entitlement, and access are three things.** A payment is a fact about the provider. An entitlement is a fact about your product. Access is a function of entitlement and current time. Collapsing them causes most billing bugs.

## Fulfillment

Make it a function of state, not of the event:

```
fulfill(order):
  if order.fulfilled_at is not null: return       # no-op
  transaction:
    order.fulfilled_at = now
    entitlement.upsert(user, product, expires_at)
  after commit: enqueue receipt
```

The database enforces it. Unique constraint on the entitlement, or on the session id. Application-level checks lose the race.

## Reconciliation

A daily job listing provider sessions from the last few days with a paid status, asserting a matching fulfilled order locally, fulfilling and alerting on any drift.

This is the step everyone skips and the one that saves you. Assume some webhook will be lost, because eventually one will be.

```
pnpm reconcile --dry-run    # reports drift, changes nothing
pnpm reconcile              # fulfills drift, idempotent, safe to run twice
```

## Edge cases that must be handled

| Case | Correct behavior |
|---|---|
| Tab closed after payment, before redirect | Webhook fulfills. Success page is not required |
| Event delivered twice | Second is a no-op via the dedupe table |
| Event arrives before the local order commits | Create the local order before redirecting to checkout |
| Two tabs, two sessions, both pay | Detect the second, refund it, alert |
| Same product bought twice legitimately | Decide: extend access, or block at session creation. Block is simpler |
| Payment succeeds, fulfillment throws | Return non-2xx so the provider retries. Never swallow |
| Refund after access is used | Policy encoded in the handler, not in support replies |
| Chargeback | Revoke immediately and alert |
| Anonymous checkout, account created after | Store against the email, claim on signup |
| Price changed mid-session | Amount is locked to the session. Verify paid amount against expectation before fulfilling |
| Test key in production, or live key in dev | Preflight asserts the prefix matches `NODE_ENV` and refuses to boot |

## Gate

```
pnpm test:e2e -- checkout                    # full test matrix
stripe trigger checkout.session.completed    # run twice, second is a no-op
pnpm reconcile --dry-run                     # zero drift
```

Plus the manual check that catches the biggest class of bug: pay, then close the tab immediately, before any redirect. The entitlement still lands.
