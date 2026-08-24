---
name: db-and-migrations
description: "Design schema, write migrations, and write queries. Use for anything touching the database: adding a table or column, changing a type, writing a migration, adding an index, writing or optimizing a query, setting up row level security policies, seeding, connection pooling, or diagnosing slow queries and connection exhaustion. Triggers include 'schema', 'migration', 'add a column', 'query', 'index', 'slow', 'RLS', 'policy', 'seed', 'too many connections'. Do NOT use for deciding which database to use; that is an ADR decision made once at project-init."
---

# Database and migrations

## Schema rules

- **Timestamps over booleans.** `fulfilled_at` beats `is_fulfilled` because it debugs itself. Store `created_at` and `updated_at` on every table.
- **Enums for more than two states.** Add the enum on day one, not after the third boolean appears.
- **Foreign keys with explicit `on delete` behavior.** Cascade, restrict, or set null, chosen deliberately. The default is rarely what you want.
- **Unique constraints enforce business invariants.** One entitlement per user per product is a constraint, not an application check that loses races.
- **No nullable columns without a meaning for null.** If null and empty mean the same thing, pick one.
- **Money is integer minor units.** Never a float. Store the currency alongside it.
- **Timestamps are `timestamptz`.** Always. A naive timestamp is a bug waiting for a daylight saving transition.

## Migrations

- Every migration is reversible. Write the down and test it before the up ships.
- Additive during a deploy, destructive as a separate later step. Adding a column is safe while old code runs. Dropping one is not.
- The three-step drop: stop writing to it, deploy, then drop in a separate migration once nothing references it.
- Never edit a migration that has run anywhere real. Write a new one.
- Backfills for large tables run in batches, outside the migration, with progress that survives interruption.
- Migrations use the direct connection string, not the pooled one. They need a real session.

```
pnpm db:migrate && pnpm db:migrate:down && pnpm db:migrate   # proves reversibility
```

## Drift

Migrations and code ship on separate tracks. A host that deploys on push does not run migrations, and nothing joins the two unless you build the join. Meanwhile a local chain that resets and reseeds keeps local permanently current, so local is green while production is behind.

`pnpm db:drift` compares applied against committed and fails **both** directions. Committed but unapplied means production is behind. Applied but uncommitted means someone edited production by hand.

Two properties that matter more than the comparison itself:

- **Refuse, never skip.** No credentials means exit non-zero with a message. A drift check that silently passes when it cannot reach the database is worse than no check.
- **Assume the silent case.** A missing `add column` breaks loudly on the first query. A missing `revoke`, constraint, or policy breaks nothing at all: the old grant stays live and everything looks healthy. If your stack exposes the database directly to clients, an unapplied permission migration is a live authorization hole that no smoke test will find.

The check runs in `pnpm gate` and again against production at deploy. Passing locally proves the local database is current, which was never in question.

## Bulk writes and imports

- **Upsert by natural key. Never delete-and-reinsert** when anything references those rows. The delete cascades through the foreign key before the insert restores anything.
- `ON DELETE SET NULL` on a nullable foreign key turns a delete into silent corruption. The child rows survive, orphaned, and nothing errors. Choose `restrict` unless there is a stated reason not to.
- A trigger or guard scoped to one operation does not cover the others. A rule that fires on update is absent during delete-then-insert, which is exactly what an importer does.
- Any import that can remove rows reports counts before and after and refuses on an unexpected delta.

## Queries

- Explicit column selection. Never `select *` into anything a user sees.
- The authorization check lives in the same query that fetches. `where id = ? and user_id = ?`, not a fetch followed by an if.
- Multi-write operations that must not partially apply go in a transaction.
- N+1 is the default failure. A loop containing a query is the tell. Fix with a join or a batched `in` clause.
- Index what you filter, sort, and join on. Composite index column order follows the query's leading predicate.
- An index has a write cost. Add it for a measured read, not a suspected one.

## Row level security

Enabled on every table, default deny, policies added explicitly. It is the backstop for when application code is wrong, which it eventually will be.

The service role key bypasses it entirely. Any code path using that key is privileged and gets reviewed as such.

## Pooling

Serverless plus Postgres exhausts connections quickly and looks exactly like slowness. Use the pooled connection string for the app and the direct one for migrations. This is not optional at any real concurrency, and it belongs in `operations.md` so the next person does not rediscover it.

## Seeding

The seed produces a database a test can rely on: deterministic ids, known users covering each state, at least one row of every entity. Tests run against a real seeded database, because mocking your own data layer tests the mock.

A seed that can touch production refuses by default and passes only through layered checks, every one of which must be explicit:

| Refusal | Reason |
|---|---|
| Not on the expected branch | You are seeding from work in progress |
| Working tree dirty | What runs is not what is committed |
| `pnpm gate` not passing | Seeding broken content is still seeding |
| Host does not match the expected target | The most common accident |
| No `--i-mean-it` intent flag | Nothing destructive happens by autocomplete |

Any one failing exits non-zero and names which. Never a single confirmation flag, because a single flag becomes muscle memory.

## Diagnosing slow

1. Slowest queries by total time, not by count.
2. Read the query plan. A sequential scan on a filtered column is the usual answer.
3. Check connection count before concluding it is a query problem.
4. Only after those three, consider caching. Caching before measuring hides the bug instead of fixing it.
