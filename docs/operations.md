# Operations

> Structure pre-filled, values filled during phases 0 and 7.
> Written to be read at 2am by someone who did not write the code. That someone is you, in six months.

## Environment variables

Every variable here appears in `src/lib/env.ts` and `.env.example`. If it does not, one of the three is wrong.

| Variable | Scope | Required | Set where | Owner | Notes |
|---|---|---|---|---|---|
| `NODE_ENV` | server | yes | platform | platform | |
| `APP_URL` | both | yes | platform | you | Absolute, no trailing slash. Redirect URLs depend on it |
| `DATABASE_URL` | server | yes | platform | provider | Pooled connection string, not direct |
| `DATABASE_URL_DIRECT` | server | migrations only | platform | provider | Unpooled. Migrations need a real session |
| `AUTH_SECRET` | server | yes | platform | you | 32 bytes minimum. Rotating invalidates all sessions |
| `STRIPE_SECRET_KEY` | server | yes | platform | you | Prefix must match `NODE_ENV`. Preflight asserts this |
| `STRIPE_WEBHOOK_SECRET` | server | yes | platform | Stripe | Different per endpoint. Test and live are not interchangeable |
| `STRIPE_PRICE_*` | server | yes | platform | you | Price ids, not product ids |
| `RESEND_API_KEY` | server | yes | platform | you | |
| `EMAIL_FROM` | server | yes | platform | you | Domain must be verified or delivery fails silently |
| `INDEXING_ENABLED` | server | no | platform | you | Defaults to noindex when unset. Deliberate |
| `LOG_LEVEL` | server | no | platform | you | `info` in production |
| `DATABASE_URL_DRIFT` | script | yes | local and CI | you | Read access for `pnpm db:drift`. Absent means the check refuses, never skips |
| <!-- FILL --> | | | | | |

## Provider settings

Configuration living in a vendor dashboard, invisible to the repo and to every test. A provider default is not your default.

| Setting | Provider | Expected value | Asserted by | Verified |
|---|---|---|---|---|
| Verification code length | auth | <!-- FILL --> | preflight | |
| Redirect URL allowlist | auth | every environment's callback | manual, per environment | |
| Email template bodies | auth | <!-- FILL --> | manual | |
| Sending domain verification | email | verified, SPF and DKIM | preflight | |
| Webhook endpoint and secret | payments | one per environment, separate secrets | preflight | |
| <!-- FILL --> | | | | |

Each row is checked at Phase 7 and again after any provider change. These do not appear in a diff, so nothing else will catch them.

Secrets live only in the platform's environment settings. Never in the repo, never in a shell history, never pasted into a chat.

## Endpoints

| Path | Question it answers | Expected | Used by |
|---|---|---|---|
| `/api/health` | Is the process alive | 200, no dependency checks | Platform liveness |
| `/api/ready` | Can it serve traffic | 200 with database and migration state, 503 otherwise | Platform readiness, uptime monitor |

Conflating these causes restart loops: a database blip fails liveness, the platform restarts a healthy process, and the restart does not fix the database.

## Runbook

### The site is down

1. `/api/health`. No response means the platform or the deploy. Response means the app is up and something else is wrong.
2. `/api/ready`. 503 names the failing dependency in the body.
3. Platform status page and deployment log. Most outages are the last deploy.
4. Database provider status and connection count. Exhausted pool looks exactly like a down database.
5. If the last deploy is the cause, roll back first and diagnose after.

### Payments are not granting access

1. Provider dashboard, webhook delivery log. Failures and their response codes are listed there.
2. `stripe_events` table. Present with a `processed_at` means it was handled and the bug is downstream. Absent means it never arrived.
3. `pnpm reconcile --dry-run` reports drift between the provider and local orders.
4. `pnpm reconcile` fulfills the drift. It is idempotent, so running it twice is safe.
5. If the webhook secret was rotated, the endpoint has been silently failing signature checks since. Check that before anything clever.

### Email is not arriving

1. Provider send log. Delivered means it is spam filtering, not sending.
2. Sending domain verification, SPF, DKIM, DMARC.
3. Confirm `EMAIL_FROM` matches a verified domain. A mismatch fails quietly.

### Migrations have drifted

1. `pnpm db:drift` against production. It reports direction, not just a boolean.
2. **Committed but unapplied** means production is behind. Check what is missing before applying. A missing `add column` has been breaking loudly already. A missing `revoke`, policy, or constraint has been breaking nothing, which means the unsafe state has been live for however long the drift has existed. Treat permission drift as an incident, not a chore.
3. **Applied but uncommitted** means someone changed production by hand. Capture it as a migration before touching anything else, or the next deploy reverts it.
4. Once clean, find the join that was missing. A host that deploys on push does not run migrations. If the deploy does not run them and nothing checks, this recurs.

### The database is slow

1. Slowest queries by total time, not by count.
2. Missing index on a newly filtered column is the usual answer.
3. Connection pool saturation. Serverless plus an unpooled connection string exhausts a database quickly and looks like slowness.

## Deploy

1. Merge to `main`.
2. Migrations run before the new code serves traffic. Additive migrations only during a deploy; destructive changes are a separate, deliberate step after the code that used the column is gone.
3. Platform builds and swaps.
4. `/api/ready` returns 200.
5. Smoke set: log in, load a protected page, load a public page.

## Rollback

Trigger a rollback when: error rate crosses <!-- FILL -->, `/api/ready` fails after a deploy, or payments stop granting access.

1. Redeploy the previous build from the platform. Fastest path, always available.
2. Migrations do not roll back with the code. An additive migration is safe to leave in place. A destructive one is why destructive migrations are separate.
3. Verify with the smoke set.
4. Write what happened in `status.md` before diagnosing. You will forget the sequence.

## Preview environments

Preview and ephemeral deployments never inherit production credentials. Each gets its own database, keys, and sending domain, or preview deployments are disabled outright.

The failure this prevents: a throwaway branch deployment with write access to real customer data, which looks like a working preview right up until it is not.

| Environment | Database | Keys | Status |
|---|---|---|---|
| Preview | <!-- FILL: own instance, or disabled --> | Test | |

## Backups

| Item | Frequency | Retention | Restore tested |
|---|---|---|---|
| Database | <!-- FILL --> | | |
| Uploads | | | |

An untested backup is not a backup. Restore once, to a scratch database, and record the date here.

## Monitoring

| Signal | Threshold | Where it goes |
|---|---|---|
| Uptime `/api/ready` | 2 consecutive failures | <!-- FILL --> |
| Error rate | | |
| Webhook handler failure | any | This one is separate on purpose. A silent webhook failure is a customer who paid and got nothing |
| p95 response time | | |

## Incident procedure

1. Contain. Revoke the credential, disable the endpoint, roll back the deploy. Diagnosis comes second.
2. Assess. What data, how many users, what window.
3. Record. Timeline in `status.md` while it is fresh, not after.
4. Notify, if user data was exposed. Know the obligation before you need it.
5. Fix, with a regression test.
6. Write what allowed it. Not who.

## Review cadence

| Check | When |
|---|---|
| Dependency audit | Every CI run |
| Authorization matrix against the code | Every phase gate from 3 onward |
| Secret scan of full history | Before first deploy, then quarterly |
| Access review, who has what | Quarterly |
| Restore test | Quarterly |

## Costs

| Service | Plan | Monthly | Scales with |
|---|---|---|---|
| <!-- FILL --> | | | |

## Revision log

| Date | Change | Why |
|---|---|---|
| | Created from template | |
