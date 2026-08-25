# Architecture

> Mostly pre-filled. It describes the template's own decisions. `project-init` fills the database and auth choice and any project-specific sections.
> Every material decision gets an ADR in `docs/adr/`. This file is the index and the standing picture.

## Stack

| Layer | Choice | Rationale |
|---|---|---|
| Framework | Next.js, App Router | Server Components remove most client-side data plumbing. One deploy target |
| Language | TypeScript, strict | Non-negotiable at this scale of solo maintenance |
| Runtime | Node 22 LTS | Pinned in `.nvmrc` and engines |
| Package manager | pnpm | Faster installs, strict node_modules, no phantom dependencies |
| Styling | Tailwind + shadcn/ui, all through tokens | You own the component source. No vendor upgrade treadmill |
| Database | Postgres <!-- FILL: Supabase or Neon --> | Relational data with real constraints. Both options are managed Postgres |
| ORM | Drizzle | SQL-shaped, no runtime engine, migrations are readable diffs |
| Auth | <!-- FILL: Supabase Auth or Better Auth --> | Supabase Auth if using Supabase, since RLS integration is the point. Better Auth otherwise |
| Payments | Stripe | Hosted Checkout. Nothing custom touches card data |
| Email | Resend | Transactional only |
| Testing | Vitest + Playwright | Unit and integration, plus e2e on money and auth paths |
| Hosting | Vercel | Matches the framework. Revisit if the job runner grows |

## The layering rule

Three layers, one direction. This is the load-bearing decision in the whole repo.

```
transports  →  src/server/  →  src/core/
(app, actions,   (DAL: authz,     (pure domain:
 webhooks,        db access,       no framework,
 cron, worker)    DTOs out)        no I/O)
```

- `src/core/` imports nothing from Next, the database, or the network. Data in, data out. This is the layer worth unit testing, and it is testable because it is pure.
- `src/server/` owns authorization. Every function authenticates, checks ownership, selects explicitly, and returns a DTO. Marked `import "server-only"` so a client import is a build error.
- Transports are thin. Parse input, call `src/server/`, shape the response. No logic.

The payoff: adding a background worker later means adding one entry file, not refactoring. Nothing below the transport layer knows or cares what invoked it.

## Feature slicing

```
src/features/<name>/
  components/   UI for this feature
  core/         pure logic for this feature
  server/       DAL and actions for this feature
  types.ts
```

Features never import each other. If two features need the same thing, it moves down into `src/core/`, `src/server/`, `src/lib/`, or `src/components/ui/`. Enforced by an ESLint import-boundary rule, not by discipline.

## Data model

<!-- FILL: entity list and relationships. Keep it here in prose plus a diagram; the schema itself is the source of truth in src/db/schema.ts -->

Standing tables the template ships regardless of project:

| Table | Purpose |
|---|---|
| `users` | Identity, owned by the auth provider |
| `orders` | Local mirror of Stripe payment state |
| `entitlements` | What a user is allowed to access, and until when |
| `stripe_events` | Webhook dedupe. Primary key is the Stripe event id |
| `audit_log` | Who did what to which record, append only |

Delete behavior is chosen deliberately per foreign key. `ON DELETE SET NULL` on a nullable column is the dangerous default: the child rows survive as orphans and nothing errors, so a bad import corrupts data silently. Prefer `restrict` unless there is a stated reason.

Separation that matters: payment, entitlement, and access check are three different things. Collapsing them is the root of most billing bugs. A payment is a fact about Stripe. An entitlement is a fact about your product. Access is a function of an entitlement and the current time.

## External services

| Service | Used for | Failure mode | Mitigation |
|---|---|---|---|
| Stripe | Checkout, webhooks | Webhook lost or delayed | Daily reconciliation job compares Stripe against local orders |
| <!-- FILL: DB --> | Everything | Connection exhaustion from serverless | Connection pooler, required not optional |
| Resend | Transactional email | Send fails silently | Failures logged and surfaced, never swallowed |
| <!-- FILL --> | | | |

## Work that does not fit a request

The escalation ladder. Do not skip rungs, and do not reach for rung 3 because a project "feels big".

| Rung | Mechanism | Use when |
|---|---|---|
| 0 | `after()` post-response | Fire and forget under the duration cap: receipts, cache warming |
| 1 | Managed job runner calling back into routes | Scheduled sends, retries, fan-out, drip sequences |
| 2 | `worker/` in this repo, deployed separately | Persistent connections, CPU-heavy work, no vendor in the loop |
| 3 | Separate API service | Multiple first-class clients, a public API, non-TypeScript workloads |

Current rung: **0**. Moving up gets an ADR.

## Security boundaries

Five layers, because any one of them will eventually be wrong.

| Boundary | Enforced by |
|---|---|
| Authenticated | `src/server/` session check |
| Authorized | Ownership check inside the same query |
| Database | Row level security policies as a backstop |
| Secrets | `src/lib/env.ts` only, server-only modules |
| Input | Schema parse at every transport |

## Threat model

Who realistically attacks a small SaaS, in order of likelihood. Defending against the last row while ignoring the first is a common and expensive mistake.

| Actor | Wants | Realistic method | Primary defense |
|---|---|---|---|
| Automated scanner | Any known vulnerability | Mass scanning for default paths, exposed keys, stale dependencies | Dependency audit, no secrets in the repo, no debug endpoints in production |
| Credential stuffer | Accounts, reusable passwords | Leaked password lists against your login | Rate limiting, lockout, no enumeration signal |
| Curious paying user | Other users' data | Editing an id in a URL or a request body | Ownership check inside every query, verified by a cross-user test |
| Freeloader | Paid features without paying | Calling an action directly, or replaying a client-side gate | Entitlement checked server side, never in the component |
| Payment fraudster | Goods without payment, or a chargeback | Stolen cards, then a dispute | Provider risk rules, dispute handler, revoke on chargeback |
| Spammer | Your domain's reputation | Signup flood, contact form abuse | Rate limits, email verification before any send |

Not in scope at this size: targeted attackers, insider threat, nation-state. Say so explicitly so effort goes where the risk actually is.

## Authorization matrix

Filled as features ship. Every row is a test, not a claim.

| Resource | Action | Who may | Enforced in | Test |
|---|---|---|---|---|
| Own profile | read, update | owner | `server/users.ts` | |
| Other profile | read | nobody | same query | cross-user 404 |
| Order | read | owner | `server/orders.ts` | |
| Entitlement | read | owner | `server/entitlements.ts` | |
| Paid content | read | owner with an unexpired, unrevoked entitlement | `server/access.ts` | |
| <!-- FILL --> | | | | |

The rule that catches the most: authentication is not authorization. Confirm the caller owns the row, in the same query that fetches it, every time.

## Data handling

| Data | Classification | Stored | Retention | Deletion |
|---|---|---|---|---|
| Email | personal | database | account lifetime | on account deletion |
| Password | credential | provider only, hashed | never in your database | n/a |
| Payment method | financial | provider only, never yours | n/a | n/a |
| <!-- FILL --> | | | | |

Card data never touches your servers. Hosted Checkout exists precisely so that stays true.

## Enforcement

A standard that is not enforced is a suggestion.

| Standard | Enforced by | Runs in |
|---|---|---|
| Strict types, no `any` | `tsc --noEmit` | `pnpm gate` |
| Layer import boundaries | ESLint import rule | `pnpm gate` |
| No literal colors, fonts, sizes in components | ESLint custom rule | `pnpm gate` |
| No `process.env` outside its module | ESLint restricted syntax | `pnpm gate` |
| Dead exports and unused files | knip | `pnpm gate` |
| Migration drift, both directions | `pnpm db:drift` | `pnpm gate`, and again at deploy |
| Secrets | gitleaks pre-commit, gate, plus periodic full history | Three layers |
| Dependency vulnerabilities | `pnpm audit` | `pnpm gate` |
| Static analysis | Semgrep | `pnpm gate` |
| Accessibility | axe in Playwright | `pnpm gate` |
| Performance budgets | Lighthouse against a production build | `pnpm gate` |
| Commit format | commitlint | Pre-commit |
| Human judgment | Mohmed's merge | Always. The last gate and the only one not automated |

CI runs `pnpm gate` and nothing else. A separate CI list would drift from the local one, and then green would mean two different things.

## Environments

| Environment | Database | Keys | Rule |
|---|---|---|---|
| Local | Local or a scratch instance | Test | Resets and reseeds freely |
| Preview | Its own instance, or disabled | Test | **Never inherits production credentials.** A preview pointed at production is a throwaway environment with write access to real data |
| Production | Production | Live | Written to only through migrations and guarded scripts |

If preview isolation cannot be guaranteed, disable preview deployments rather than trusting them.

## Provider settings

Configuration that lives in a vendor dashboard rather than the repo: token and code lengths, email template bodies, redirect URL allowlists, webhook endpoints, sending domain verification, plan limits. The repo cannot see any of it and a provider default is not your default.

Every setting the code depends on has a row in the table in `operations.md`, and a preflight assertion wherever the provider exposes one.

## Decisions that are deliberately deferred

| Deferred | Until | Why not now |
|---|---|---|
| Caching layer | A measured p95 problem | Premature caching hides correctness bugs |
| Multi-tenancy | A second customer type exists | Retrofitting is cheaper than guessing wrong |
| i18n | A non-English market is real | The abstraction costs more than it returns at zero users |
| Separate API service | Rung 3 conditions are met | See ladder above |
| Skills distributed independently of the template | The same skill fix has been copied into a third project | A template is a snapshot and that is fine for code. Splitting `.claude/` into its own marketplace before there is a third consumer is maintenance for an audience of one |

## ADR index

| ID | Decision | Status | Date |
|---|---|---|---|
| 0001 | Three-layer boundary: core, server, transport | Accepted | |
| 0002 | One gate command, CI calls it | Accepted | |
| 0003 | <!-- FILL: database choice --> | | |
| 0004 | <!-- FILL: auth choice --> | | |

## Revision log

| Date | Change | Why |
|---|---|---|
| | Created from template | |
