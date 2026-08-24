# Event map

The provider emits dozens of event types. Roughly eight change state in a normal app. Subscribe to what you handle and ignore the rest, because a handler you wrote and forgot is worse than no handler.

## One-time purchase

| Event | Means | Handler does |
|---|---|---|
| `checkout.session.completed` | Session finished | Fulfill if payment status is paid, otherwise mark pending |
| `checkout.session.async_payment_succeeded` | Delayed method cleared | Fulfill |
| `checkout.session.async_payment_failed` | Delayed method failed | Notify, keep locked |
| `checkout.session.expired` | Abandoned | Mark abandoned, optional recovery mail |
| `charge.refunded` | Refund issued | Revoke per policy |
| `charge.dispute.created` | Chargeback | Revoke, alert yourself |
| `charge.dispute.closed` | Resolved | Restore or finalize |
| `payment_intent.payment_failed` | Attempt failed | Log only, usually no state change |

## Subscription, in addition

| Event | Means | Handler does |
|---|---|---|
| `customer.subscription.created` | Started | Create entitlement with period end |
| `customer.subscription.updated` | Plan change, renewal, reactivation, cancel-at-period-end | Sync entitlement to the subscription's current state |
| `customer.subscription.deleted` | Ended | Revoke at period end, not immediately, unless it was an immediate cancel |
| `invoice.payment_succeeded` | Renewal paid | Extend entitlement |
| `invoice.payment_failed` | Renewal failed | Start dunning. Do not revoke on the first failure |
| `customer.subscription.trial_will_end` | Three days out | Notify |

For subscriptions, prefer syncing from the subscription object over accumulating deltas from events. Events arrive out of order. The object is always current.

## Delayed payment methods

Bank debits and transfers are not instant. The session completes with an unpaid status and the payment resolves later. If you gate fulfillment on `completed` alone, you hand out access for money that never arrives.

Either handle the async events, or disable those payment methods explicitly at session creation. Both are valid. Silently assuming instant payment is not.

## Handler shape

```
POST /api/webhooks/stripe
  read raw body                          # no parser in front
  verify signature                       # 400 on failure, do not retry
  insert event id into stripe_events     # unique constraint
    already present -> return 200        # no-op
  handle(event)                          # state writes only, fast
    throws -> mark failed, return 500    # provider retries
  return 200
```

Signature failure is a 400 because retrying will not help. Handler failure is a 500 because retrying might.
