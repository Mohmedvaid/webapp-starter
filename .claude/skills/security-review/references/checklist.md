# Security checklist

Run the whole thing before first deploy and at every phase gate from 3 onward. Between gates, run the sections a change actually touches.

## Authorization

- [ ] Every parameterized route has a cross-user test: user A requests user B's resource, expects 404
- [ ] Ownership is checked inside the query, not fetched then compared
- [ ] Server actions re-authenticate and re-authorize. Being called from your own UI is not a check
- [ ] Row level security enabled on every table, default deny
- [ ] Every service-role code path is listed and justified. That key bypasses RLS entirely
- [ ] Admin surfaces check a role, not just a session
- [ ] The authorization matrix in `architecture.md` matches the code, row by row

## Secrets

- [ ] `process.env` appears only in `src/lib/env.ts`
- [ ] No secret in a variable exposed to the client bundle
- [ ] Layered scanning: pre-commit hook, `pnpm gate`, and a periodic full-history sweep. One layer is a layer that gets bypassed
- [ ] Logger redacts tokens, keys, passwords, payment details by default
- [ ] Rotation procedure exists for each secret in `operations.md`
- [ ] Any secret that ever reached a client is treated as compromised and rotated, not just removed

## Input

- [ ] Every boundary parses with a schema: bodies, search params, route params, form data, webhooks, third-party responses
- [ ] Uploads are size-limited, type-checked by content not extension, and stored outside the web root
- [ ] User content is escaped on output. Any raw HTML rendering is sanitized and justified in a comment
- [ ] Redirect targets are validated against an allowlist. Open redirect is a phishing gift
- [ ] Queries are parameterized. No string concatenation into SQL

## Authentication

- [ ] No enumeration signal on signup, login, or reset
- [ ] Tokens single-use and expiring, consumption invalidates
- [ ] Rate limits on login, reset request, and verification resend
- [ ] Session cookies are HTTP-only, secure, SameSite
- [ ] Password reset session policy decided and documented
- [ ] Account lockout has a documented unlock path that is not "contact support and hope"

## Payments

- [ ] Webhook signature verified against the raw body
- [ ] Event dedupe with a unique constraint
- [ ] Fulfillment idempotent, enforced by the database
- [ ] Nothing fulfills on the success URL
- [ ] Key prefix asserted against `NODE_ENV` at boot
- [ ] Reconciliation job scheduled and alerting
- [ ] Refund and dispute handlers exist and revoke correctly

## Transport and headers

- [ ] HTTPS enforced, HSTS set
- [ ] CSP without `unsafe-inline`
- [ ] `frame-ancestors` none unless embedding is a feature
- [ ] `X-Content-Type-Options: nosniff`
- [ ] Referrer policy set
- [ ] CORS is not a wildcard on anything authenticated

## Dependencies

- [ ] `pnpm audit` clean at moderate and above
- [ ] Lockfile committed, `ci` used in deploys not `install`
- [ ] Any accepted vulnerability has a dated exception in `architecture.md`
- [ ] New dependencies justified: why not twenty lines of your own code

## Environments

- [ ] Preview and ephemeral deployments do not inherit production credentials. This is the single most common way a throwaway environment writes to real data
- [ ] Every non-production environment points at its own database, its own keys, and its own sending domain
- [ ] If isolation cannot be guaranteed, preview deployments are disabled rather than trusted
- [ ] Provider dashboard settings your code depends on are recorded in `operations.md` and asserted where the provider allows it

## Migrations and bulk writes

- [ ] `pnpm db:drift` clean against production, failing both directions
- [ ] The drift check refuses rather than skips when credentials are absent
- [ ] Permission, policy, and constraint migrations verified as applied, not assumed. They fail silently when missed
- [ ] No importer deletes and reinserts rows that anything references
- [ ] Nullable foreign keys reviewed for `ON DELETE SET NULL`, which corrupts silently
- [ ] Production write scripts refuse on branch, dirty tree, failing gate, wrong host, and missing intent flag

## Data

- [ ] Card data never touches your servers
- [ ] Personal data inventory current in `architecture.md`
- [ ] Deletion actually deletes, or the retention is documented and lawful
- [ ] Backups exist, and a restore has actually been performed once

## Operational

- [ ] Error responses leak nothing: no stack traces, no internal names, no SQL
- [ ] Debug endpoints and verbose logging disabled in production
- [ ] Webhook failure has its own alert, separate from general error rate
- [ ] Rollback procedure documented and rehearsed once
- [ ] Every guard has a test asserting it fires at its call site, not only that its logic is correct
