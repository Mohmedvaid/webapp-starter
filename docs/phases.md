# Phases

> Pre-filled. The phases and gates are the template's. Only the feature content inside Phase 3 changes per project.
>
> A gate is passed by running the command and reading the output, not by believing it passed. A phase with a skipped test, a suppressed type error, or an unhandled case is not complete. Say so instead of moving on.

## How phases work

- Sequential. Phase N+1 does not start until phase N's gate passes.
- Each phase ends with a demo. If it cannot be shown in under two minutes, it was scoped wrong.
- `status.md` records the current phase and what blocks it, updated at the end of every session.

## The gate

Every phase runs the same command:

```
pnpm gate
```

Typecheck, lint, dead-export check, unit, e2e, production build, migration drift, secret scan. CI runs this and nothing else, so local green and CI green mean the same thing.

Each phase adds one **adversarial check** on top. That check is not a repeat of the gate. It is the thing designed to catch the specific way this phase lies about being finished, and it is the part worth pasting into the PR.

---

## Phase 0: Foundation

Boots, refuses to boot when misconfigured, gate is green before a feature exists.

**Deliverables.** Zod env schema parsed at import, the only reader of `process.env`. Preflight: database reachable, migrations current, key mode matches environment, webhook secret present, provider settings asserted. `/api/health` and `/api/ready` as separate endpoints. Result type, error taxonomy, structured logger with request id and default redaction. Middleware: security headers, request id, rate limit hook. `pnpm db:drift` comparing applied against committed, both directions, refusing when credentials are absent. `pnpm gate` wired. Lint rules: import boundaries, no `process.env` outside its module, no literal design values. gitleaks, knip, commitlint, Prettier. CI calling `pnpm gate`.

**Adversarial check.** Delete a required variable from the environment and start the app. It must refuse and name the variable. Then point the drift check at nothing and confirm it exits non-zero rather than passing.

**Demo.** Boot it, break the config, watch it refuse and say why.

---

## Phase 1: Design system and shell

**Deliverables.** Token contract in one file, Tailwind reading only variables. Component inventory. App shell, navigation, skip link, theme toggle. Route-level loading, error, and not-found. Dark mode by root token swap. axe wired into Playwright.

**Adversarial check.** Grep components and features for hex codes and raw px. Zero results, or the lint rule has a hole worth fixing. Change one token value and confirm the whole app changes.

**Demo.** Retheme live. Tab a page start to finish.

---

## Phase 2: Authentication

**Deliverables.** Every branch in `auth-flows/references/flow-trees.md`, built or explicitly declined in `product.md`. Server, browser, and middleware clients kept distinct. `requireUser` and `requireOwner` in `src/server/`. Rate limits on login, reset, and resend. Row level security enabled, default deny. Redirect to intent.

**Adversarial check.** Authenticate as user A, request user B's resource by id, on every parameterized route. Expect 404. If it passes trivially, confirm the route exists before believing it.

**Demo.** Sign up, verify, reset, log in. Then each failure branch, handled.

---

## Phase 3: Data layer and reference slice

**Deliverables.** Schema with timestamps over booleans, enums for multi-state, explicit delete behavior, integer minor units for money. Reversible migrations. Pooled connection for the app, direct for migrations. Deterministic seed, with production refusals layered on branch, clean tree, gate, host, and intent flag. Data access with authorization inside the query and DTOs out. One vertical feature slice: components, core, server, types. Unit tests on pure logic with no mocks, integration against a real seeded database.

**Adversarial check.** Import from `src/server/` into a client component and confirm a build error. Run the seed against the wrong host and confirm it refuses. Delete a parent row and confirm nothing referencing it was silently orphaned.

**Demo.** The feature working, then the same operation as another user, refused.

---

## Phase 4: Payments

**Deliverables.** Session creation with an idempotency key, local order written before redirect. Webhook route: raw body, signature verified, event id deduped by unique constraint, fast 2xx. Fulfillment as a function of state, transactional, database-enforced. The eight handlers. Entitlement separate from payment, access as a pure function of entitlement and time. Daily reconciliation with a dry-run mode. Delayed payment methods handled or explicitly disabled.

**Adversarial check.** Pay, then close the tab before any redirect. The entitlement still lands. Deliver the same event twice and confirm the second is a no-op with no duplicate entitlement.

**Demo.** Buy, close the tab early, show access granted anyway. Then a decline, handled.

---

## Phase 5: Hardening

**Deliverables.** Request id correlated end to end, redaction verified. Error taxonomy applied, nothing raw reaching a user, 404 rather than 403 for another user's resource. Rate limits on every mutation. CSP without `unsafe-inline`, HSTS, frame-ancestors, nosniff, referrer policy. Error monitoring with a dedicated webhook-failure alert, separate from the general error rate. Semgrep in the gate. The authorization matrix in `architecture.md` verified row by row against the code.

**Adversarial check.** Trigger one handled and one unhandled error. Confirm the user sees something useful, the log carries the request id, and the monitor fired. A matrix row with no corresponding test is a claim, not a control.

**Demo.** A failure traced from the screen to the log line to the alert.

---

## Phase 6: Public surface

**Deliverables.** Per-route metadata, Open Graph, canonical URLs. Sitemap and robots gated by an indexing flag that **defaults to noindex when unset**. Marketing and pricing pages matching `product.md`. Legal pages, marked placeholder until reviewed. Support contact reaching a real inbox. Analytics with no personal data in payloads. Lighthouse wired into the gate, asserting the budgets declared in `product.md`.

**Adversarial check.** Unset the indexing variable entirely and confirm robots still disallows. Run Lighthouse against a production build, not the dev server, and confirm it fails when a budget is breached rather than merely reporting a number.

**Demo.** Public pages, the flag both ways, and a budget deliberately broken to prove the gate bites.

---

## Phase 7: Deploy

**Deliverables.** Production environment set and verified against the schema. Migrations applied and verified, not assumed. Live payment mode, production webhook endpoint with its own secret. Preview deployments either isolated from production credentials or disabled. Backups running, and a restore actually performed once. Runbook complete. Rollback documented and rehearsed.

**Adversarial check.** The production smoke set, run against production: sign up, verify, log in, reset, protected page, public page, readiness, one real purchase then refunded. Then `pnpm db:drift` against production, and a data-state assertion. A production database holding a fraction of what git holds returns 200 on every check while being wrong.

**Demo.** The smoke set, on production, in front of yourself. Environment-only failures are the norm.

The phase ends with a deployed but unlaunched app; going live runs the launch checklist.

---

## Phase status

| Phase | Status | Gate passed | Adversarial check | Notes |
|---|---|---|---|---|
| 0 Foundation | Not started | | | |
| 1 Design system | Not started | | | |
| 2 Auth | Not started | | | |
| 3 Data + slice | Not started | | | |
| 4 Payments | Not started | | | |
| 5 Hardening | Not started | | | |
| 6 Public surface | Not started | | | |
| 7 Deploy | Not started | | | |

Paste real output. A ticked box with no output is the failure mode this column exists to prevent.
