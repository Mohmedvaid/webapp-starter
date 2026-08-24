# Payment test matrix

Five success cases and one decline case is a demo, not a test plan. Production breaks in the branches.

## Cards

Test mode only. Any future expiry, any CVC, any postal code.

| Number | Outcome |
|---|---|
| 4242 4242 4242 4242 | Succeeds |
| 4000 0000 0000 0002 | Generic decline |
| 4000 0000 0000 9995 | Insufficient funds |
| 4000 0000 0000 0069 | Expired card |
| 4000 0000 0000 0127 | Incorrect CVC |
| 4000 0025 0000 3155 | 3D Secure challenge required |
| 4000 0084 0000 1629 | 3D Secure passes, then declines |
| 4000 0000 0000 0259 | Succeeds, then disputed as fraudulent |

The 3DS-then-decline card is the one that finds half-built flows, because the UI often treats a passed challenge as a completed payment.

API-level equivalents exist as `pm_card_*` payment method tokens. Adding a `+location_XX` suffix to a test email simulates a customer's country, which changes the payment methods offered.

## Webhook scenarios

Run each with the CLI, not just in the happy-path e2e:

| Scenario | Expected |
|---|---|
| `stripe trigger checkout.session.completed` | Fulfills once |
| Same event delivered twice | Second is a no-op, no duplicate entitlement |
| Handler throws | Non-2xx returned, provider retries, eventual success |
| Bad signature | 400, nothing written, no retry |
| Event for an unknown local order | Handled without crashing, alerted |
| Refund event | Entitlement revoked per policy |
| Dispute event | Entitlement revoked, alert fired |

## Manual checks

Automation misses these. Run them by hand before the phase 4 gate.

1. Pay, then close the tab before any redirect. Access still granted.
2. Open checkout in two tabs, pay in both. Second payment detected and refunded.
3. Rotate the webhook secret without updating the env. Confirm it fails loudly rather than silently dropping events.
4. Run with a live key while `NODE_ENV` is development. Confirm preflight refuses to boot.

## Live mode

Test and live are separate environments with separate keys, separate customers, and separate webhook endpoints. Nothing crosses.

Before the first real charge: register the production webhook endpoint, store its own secret, and run one real purchase with a real card followed by an immediate refund. Test mode passing tells you nothing about a misconfigured production endpoint.
