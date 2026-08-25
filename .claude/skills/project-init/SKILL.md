---
name: project-init
description: "Turn the webapp-starter template into a specific project. Use on the first session in a freshly cloned repo, or when Mohmed says 'set this up', 'new project', 'initialize', 'let's start', or asks what to do first in an unconfigured repo. Runs a short interview, records the database and auth choice as ADRs, fills the FILL markers across the docs folder, deletes the payment mode that is not being used, and leaves the repo ready for the Phase 0 gate. Do NOT use on a repo that already has a filled docs/product.md; that project is already initialized and the work belongs to phase-planning instead."
---

# Project init

Runs once, at the start. Ends with a repo that is about a specific product rather than a template.

This skill runs in a repo created from the template, not a fork. History is fresh and there is no upstream link. Template improvements made after creation do not arrive automatically: the docs and skills in the clone are a snapshot from creation time, not a live feed.

## Interview

Ask these in one message as a numbered list. Do not ask them one at a time. If Mohmed answers only some, fill what you can and list what is still missing.

1. One line: what it does, for whom.
2. Primary user, and how they solve this today.
3. The one number that says it is working.
4. Money: one-time or subscription, price, what the price anchors against.
5. Three things this explicitly does not do.
6. Database: Supabase or Neon. If he skips this, recommend Supabase when auth plus row level security plus storage are all wanted, Neon when plain Postgres is enough, and say which you picked.
7. Domain and sending domain, if known.

Do not ask about the stack beyond the database and auth. Those are decided by the template.

## Fill order

1. `docs/product.md` from answers 1 through 5. Every FILL marker gone or explicitly marked unknown.
2. `docs/architecture.md`: database row, auth row, external services table.
3. `docs/adr/0003` database choice, `docs/adr/0004` auth choice. Use the 0001 format. Short. Name the alternative and why it lost. 0001 and 0002 are template decisions and already exist.
4. `docs/operations.md`: env var table completed for the chosen services.
5. `docs/design-system.md`: token values if there is a brand, placeholder scale if not.
6. `docs/product.md`: epic headings only, no stories yet. Stories are `spec-writing`, not this.
7. `docs/product.md`: anything from the interview that was explicitly deferred.
8. `docs/status.md`: phase 0, first three Next items, last updated stamped.

## Delete

The template ships both payment modes. Delete the unused one now, not later.

- Keeping one-time: remove subscription price handling, `customer.subscription.*` handlers, and the billing portal route.
- Keeping subscription: remove the single-purchase entitlement path and its tests.

Also: rename the package, replace the README title and description, remove the Projects vertical slice only when a real feature exists, never before.

## Record the choices in code

- `src/lib/env.ts` schema matches the chosen services exactly. No orphan variables.
- `.env.example` matches the schema exactly. A variable in one and not the other is a phase 0 gate failure.
- `src/db/` adapter wired to the chosen provider, pooled connection string in `DATABASE_URL`, direct in `DATABASE_URL_DIRECT`.
- `scripts/drift.ts` adapter selected for the chosen migration tool. The comparison logic is generic; only the "list applied migrations" call changes.
- Provider settings table in `operations.md` filled for the chosen services, so the dashboard configuration the code assumes is written down somewhere a diff can show.

## Gate

Init is done when:

```
pnpm install && pnpm preflight && pnpm dev
pnpm gate
grep -rn "FILL" docs/product.md docs/architecture.md docs/operations.md
```

The grep returns nothing, or returns only markers Mohmed explicitly deferred. Then hand off to `phase-planning` for Phase 0.

## Do not

- Do not start building features. Init ends at the Phase 0 gate.
- Do not invent charter answers. An unanswered question is recorded as an open decision in `status.md`, not guessed.
- Do not add a dependency during init. The template's stack is the stack until an ADR says otherwise.
