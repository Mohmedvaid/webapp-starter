# webapp-starter

Next.js + TypeScript web app template. Cloned per project, then customized.
This file is always in context. Keep it under 120 lines. Detail belongs in a skill or a doc.

## Session protocol

1. Read `docs/status.md`. It is the only reliable statement of where things are.
2. Read the current phase in `docs/phases.md`. Do not skip a phase gate.
3. Do the work. Load the skill that matches, do not improvise a process.
4. Update `docs/status.md` before ending. Stale status is worse than no status.

## Commands

```
pnpm gate           the whole chain. This is the gate
pnpm dev            pnpm db:migrate     pnpm preflight
pnpm test           pnpm db:seed        pnpm db:drift
```

`pnpm gate` runs typecheck, lint, dead-export check, unit, e2e, production build, migration drift, and secret scan. CI runs the same command and nothing else, so a green local gate and a green CI mean the same thing.

Work is not done until `pnpm gate` passes. Say so plainly if it does not.

## Routing

| Task | Skill |
|---|---|
| New project, first run | `project-init` |
| Requirements, user stories | `spec-writing` |
| Break down work, plan a phase | `phase-planning` |
| Any implementation, any bug | `tdd-workflow` |
| Login, signup, reset, session, OAuth | `auth-flows` |
| Checkout, webhook, refund, billing | `payments-stripe` |
| Schema, migration, query, index | `db-and-migrations` |
| Env vars, health checks, logging, boot failures | `preflight-and-observability` |
| Route handler, server action, endpoint | `api-and-validation` |
| Pre-deploy or "is this safe" | `security-review` |
| Color, spacing, theme, tokens | `design-system` |
| Components, forms, data fetching | `frontend-patterns` |
| What to test, coverage, e2e | `testing-strategy` |
| Review before merge | `code-review` |
| Branch, commit, PR | `git-workflow` |
| After any shipped feature | `docs-maintenance` |
| Deploy, release, go live | `release-and-deploy` |

## Non-negotiables

Full set in `.claude/rules/`. These ten cause the most damage when broken:

1. Authorization lives in `src/server/`, not middleware. Middleware is an optimistic redirect and nothing more.
2. Every server action and route handler validates input and re-authorizes. Treat them as public POST endpoints.
3. Return DTOs. Never raw database rows.
4. Parse every external input at the boundary with a schema: request bodies, params, env, webhooks, third-party responses.
5. Payment fulfillment happens in the webhook, is idempotent, and is enforced by a database constraint.
6. No literal colors, fonts, spacing, or radii in components. Tokens only.
7. `src/core/` imports no framework and reaches for no clock, randomness, or id generator. Those are injected, which is what makes it deterministic.
8. State is recorded as timestamps, not booleans. `fulfilled_at`, not `is_fulfilled`.
9. A check that skips when it cannot run reports success falsely. Refuse instead.
10. Local green does not mean production correct. Drift, data state, and provider settings are invisible locally.

## Never

- Never commit a secret, or read `process.env` outside `src/lib/env.ts`.
- Never use `any`, or silence a type error with a cast you cannot justify in one sentence.
- Never fulfill an order on the success URL.
- Never write to production data without layered refusals: branch, clean tree, gate, host, and an explicit intent flag.
- Never assume a migration applied because the deploy succeeded. They are separate tracks unless something joins them.
- Never ship a guard without a test that proves it fires at its call site.
- Never create a new doc when an existing one covers the topic. Update it.
- Never claim a gate passed without running the command and reading the output.
- Never mark a phase complete with a failing or skipped test.

## Layout

```
src/app/            routes only, no logic
src/features/<n>/   components, core, server, types for one feature
src/core/           cross-feature pure domain, zero framework imports
src/server/         cross-feature data access, server-only, owns authz
src/components/ui/  generic, no business meaning
src/lib/            env, logger, result, errors, rate-limit, clients
src/db/             schema, migrations, seed
docs/               product, architecture, design-system, phases, operations, status, adr/
.claude/            rules, skills
```

Import direction is one way. `app` may import `features`. `features` may import `core`, `server`, `lib`, `components/ui`. Nothing imports `app`. A feature never imports another feature; if it needs to, the shared piece moves down.

## Docs ownership

| Doc | Owns |
|---|---|
| `product` | What, why, stories, acceptance criteria, MVP line, backlog |
| `architecture` | Stack, boundaries, data model, threat model, authz matrix, ADRs |
| `design-system` | Tokens, components, states, accessibility |
| `phases` | Order of work and the gates |
| `operations` | Env, runbook, rollback, incidents |
| `status` | Right now |
