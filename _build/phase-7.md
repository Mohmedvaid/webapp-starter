# Phase 7: Deploy and template completion

**Skills:** `release-and-deploy`, `security-review`, `docs-maintenance`
**Gate:** `docs/phases.md`, Phase 7

This phase has two halves. The first deploys a working app. The second finishes the template.

## Half 1: Deploy

**Platform.** Vercel project, production environment variables set and verified against the schema.

**Database.** Production instance, pooled and direct connection strings, migrations run **and verified** with `pnpm db:drift` against production. Applied is not the same as deployed.

**Preview environments.** Isolated from production credentials, or disabled. A preview pointed at production is a throwaway environment with write access to real data.

**Provider settings.** Every row in the table in `docs/operations.md` checked against the actual dashboard. None of it appears in a diff.

**Payments.** Live mode activated, production webhook endpoint registered with its own secret. Test mode passing says nothing about a misconfigured production endpoint.

**Backups.** Confirmed running, and a restore actually performed once against a scratch database. An untested backup is not a backup.

**`docs/operations.md`.** Runbook complete, every FILL filled, thresholds set.

**Rollback.** Documented and rehearsed once. Not read, rehearsed.

**Smoke set, against production.** Sign up, verify, log in. Password reset end to end. Protected page. Public page. `/api/ready`. One real purchase with a real card, then refund it. Confirm logs and alerts arriving. `pnpm db:drift` clean. A data-state assertion, since a production database holding a fraction of what git holds returns 200 on every other check.

## Half 2: Finish the template

**Reset for cloning.**

- `docs/status.md` back to its empty template state. The template's own build history is not a clone's starting status.
- `docs/product.md` FILL markers intact. If the build filled them, restore them.
- `README.md` accurate for a fresh clone.

**Verify the clone path.** Clone the repo to a scratch directory, run `pnpm install`, and confirm `project-init` has everything it needs. If a step requires knowledge that lives only in this conversation, that knowledge belongs in a doc.

**Delete `_build/`.** Per `CLEANUP.md`. The whole folder, including `CLEANUP.md` itself.

**Tag `v1`.**

## Avoid

- Deploying and calling the template done. Half 2 is the deliverable.
- Leaving the template's own status and charter content in place. The next clone reads them as its own.
- Deleting `_build/` before the phase 7 gate passes.
- Running destructive migrations during the deploy. Additive during, destructive as a separate later step.

## Gate

The production smoke set, run against production, output pasted. Then:

```
pnpm gate
git clone <repo> /tmp/clone-test && cd /tmp/clone-test && pnpm install && pnpm typecheck
ls _build 2>/dev/null || echo "build folder removed"
git tag
```
