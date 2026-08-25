---
name: release-and-deploy
description: "Ship to production and recover when a deploy goes wrong. Use when deploying, cutting a release, running migrations against production, activating live payments, turning on search indexing, or rolling back. Triggers include 'deploy', 'ship it', 'go live', 'release', 'production', 'roll back', 'the deploy broke something', 'turn on indexing', 'switch to live keys'. Load references/launch.md at go-live. Covers migration ordering, the production smoke set, rollback triggers, and the irreversible steps that need a deliberate decision. Do NOT use for the phase gates themselves; those live in docs/phases.md."
---

# Release and deploy

Environment-only failures are the norm, not the exception. Cookie domains, redirect URLs, sending domains, key modes. Everything below assumes the code is fine and the environment is where it breaks.

Load `references/launch.md` at go-live.

## Order

1. Merge to `main` after Mohmed's approval.
2. Migrations run before the new code serves traffic. **Verify your host actually does this.** Deploy-on-push hosts do not run migrations, and nothing joins the two tracks unless you build the join. Run `pnpm db:drift` against production as part of the deploy and fail it on drift.
3. Platform builds and swaps.
4. `/api/ready` returns 200.
5. Smoke set.

## Migration ordering

The rule that prevents most deploy outages: **additive during a deploy, destructive as a separate later step.**

Adding a column is safe while old code still runs. Dropping one is not, because the old code is still selecting it during the swap.

The three-step drop, across three deploys:

1. Stop writing to and reading the column. Deploy.
2. Confirm nothing references it.
3. Drop it in its own migration. Deploy.

Migrations use the direct connection string. The pooled one does not give them a real session.

Backfills on large tables run outside the migration, in batches, resumable.

## Irreversible steps

Each of these is a deliberate decision, made once, never as a side effect of a deploy:

| Step | Why it is one-way |
|---|---|
| Live payment mode | Real money, real customers, real disputes |
| Search indexing on | Crawlers cache. A half-built site indexed is a slow problem to undo |
| First customer email send | Domain reputation is earned slowly and lost quickly |
| Destructive migration | The data is gone |
| Production seed or import | Delete-and-reinsert destroys referencing rows. See the refusal list in `db-and-migrations` |

Indexing defaults to noindex when the flag is unset. Confirm that before deploy, because defaulting the other way means an unset variable publishes the site.

## Production smoke set

Run against production, every deploy that touches auth, payments, or data. By hand the first time, automated after.

- Sign up, verify, log in
- Password reset end to end
- Load a protected page
- Load a public page
- `/api/ready` returns 200
- One real purchase with a real card, then refund it, before the first release only
- Confirm logs and alerts are arriving
- **Assert data state, not just status codes.** Row counts or content parity against expectation. A production database holding a fraction of what git holds returns 200 on every check while being wrong for a week
- `pnpm db:drift` against production returns clean

Test mode passing tells you nothing about a misconfigured production webhook endpoint. That endpoint has its own secret and it is registered separately.

## Rollback

Triggers, decided in advance so nobody negotiates during an incident:

- Error rate crosses the threshold in `operations.md`
- `/api/ready` fails after a deploy
- Payments stop granting access
- Any data loss, at any rate

Procedure:

1. Redeploy the previous build. Fastest path, always available.
2. Migrations do not roll back with the code. An additive migration is safe to leave; this is exactly why destructive ones are separate.
3. Verify with the smoke set.
4. Write the timeline in `status.md` before diagnosing. The sequence is the first thing you forget.

Revert first, diagnose after.

## After a release

- Watch the error rate and the webhook alert for the first hour.
- Update `status.md` with what shipped.
- Update `phases.md` if a gate passed.
- Note anything the environment surprised you with in `operations.md`, so the next deploy does not rediscover it.
