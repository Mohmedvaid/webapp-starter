# Launch checklist

Deploying is not launching. Phase 7 ends with a working production app that nobody can find. This runs at the moment you make it findable, and most of it is irreversible.

Load `../../security-review/references/checklist.md` and pass it first. This file does not repeat it.

Order matters. Everything reversible happens before anything irreversible, and the indexing flip is last.

---

## Gates already passed

- [ ] Phase 7 gate passed, output pasted in `status.md`
- [ ] Security checklist passed in full
- [ ] `pnpm gate` green on `main`, not on a branch
- [ ] `pnpm db:drift` clean against production, both directions

## Domain

- [ ] Apex and www both resolve, one redirects to the other, and it is the same one every time
- [ ] SSL valid on both, no mixed content warnings
- [ ] DNS propagated, checked from a network that is not yours
- [ ] Canonical URLs point at the version you actually chose

## Email

The provider reporting "delivered" is not delivery.

- [ ] Test send lands in a real Gmail inbox, not spam
- [ ] Test send lands in a real Outlook inbox, not spam
- [ ] SPF, DKIM, and DMARC all pass on a received message, checked in the headers
- [ ] Every transactional template renders with real data: verification, reset, receipt, refund
- [ ] Support address round-trips: mail in arrives, and a reply from it sends and lands
- [ ] Sender name and reply-to are what a customer should see, not a no-reply placeholder

## Money truth

Each of these is a place the site can disagree with itself.

- [ ] Pricing page number matches the **live price object**, not just `product.md`
- [ ] Currency and tax handling match what the checkout actually charges
- [ ] Refund policy text matches what the refund handler actually does to an entitlement
- [ ] Terms and privacy reviewed and no longer marked placeholder
- [ ] What the customer receives after paying matches what the pricing page promised, including the access period
- [ ] One real purchase, real card, live mode: entitlement granted, receipt received
- [ ] Refund that purchase: entitlement revoked exactly as the policy states
- [ ] Both of the above visible in the provider dashboard and in your own tables

## Observability armed

Assumed alerts are not alerts.

- [ ] Error monitor receiving, verified by triggering one
- [ ] Webhook failure alert **fired**, verified by forcing a handler failure
- [ ] Uptime check running against `/api/ready`, and it has alerted once on purpose
- [ ] Logs queryable by request id, end to end, from a real request you made
- [ ] You know where the alert arrives and you will see it away from your desk

## Recovery proven

- [ ] Backup restore performed to a scratch database, with the date recorded
- [ ] Rollback rehearsed, not just written down
- [ ] Runbook readable by someone who did not write the code
- [ ] Rollback trigger thresholds set to numbers, not adjectives

## Content and data

- [ ] Production holds what git holds. Row counts asserted, not eyeballed
- [ ] No seed, demo, or test records visible to a real user
- [ ] No test-mode payment records in production tables
- [ ] Every page a crawler can reach is finished. A half-built page indexed is slow to undo

---

## The flip

In this order. Do not reorder for convenience.

1. Everything above is green
2. Set the indexing flag on and deploy
3. `robots.txt` allows, sitemap resolves and lists the right URLs
4. Submit the sitemap
5. Announce

Between step 2 and step 5 you can still reverse. After step 5 you cannot.

## First hour

- [ ] Error rate watched, not glanced at
- [ ] First real signup followed end to end, including the email
- [ ] First real purchase followed end to end, including the entitlement
- [ ] Webhook deliveries all 2xx in the provider dashboard

## First week

- [ ] `pnpm db:drift` against production, daily
- [ ] Reconciliation job ran and reported zero drift
- [ ] Data-state assertion still passing
- [ ] Anything that surprised you written into `operations.md` while you still remember it