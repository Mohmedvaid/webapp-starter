# Phase 3: Data layer and the reference slice

**Skills:** `db-and-migrations`, `api-and-validation`, `tdd-workflow`, `testing-strategy`
**Gate:** `docs/phases.md`, Phase 3

One vertical slice, end to end, proving the layering. This slice is deliberately generic and is deleted per clone. See `CLEANUP.md`.

## Build

**Schema.** Drizzle. The standing tables from `docs/architecture.md`: `users`, `orders`, `entitlements`, `stripe_events`, `audit_log`. Plus `projects` for the reference slice. Timestamps not booleans, enums for multi-state, explicit `on delete`, money as integer minor units, `timestamptz` throughout.

**Migrations.** Reversible. Test the down before the up ships. Direct connection string, not pooled.

**Pooling.** Pooled string for the app, direct for migrations. Document both in `docs/operations.md`.

**Seed.** Deterministic ids, one user in each meaningful state, at least one row of every entity. A production path exists and refuses by default: wrong branch, dirty tree, failing gate, unexpected host, or missing intent flag each exit non-zero and name which. Never one confirmation flag, because one flag becomes muscle memory.

**Import path.** If the slice imports data, it upserts by natural key. Never delete-and-reinsert. Report counts before and after and refuse on an unexpected delta.

**`src/server/`.** Data access with `requireUser` inside each function, the ownership predicate inside the same query, explicit column selection, DTO out.

**`src/features/projects/`.** `components/`, `core/`, `server/`, `types.ts`. A list, a detail view, create, update, delete. Server Actions for mutations, schema-validated, re-authorized.

**Tests.** Unit on `core/` with no mocks. Integration against a real seeded database. Factories for every entity.

## Avoid

- Fetching a row then comparing ownership in an if. The predicate goes in the query.
- `select *` into anything a user sees.
- A loop containing a query. That is N+1 and it is the default failure.
- Mocking your own database. The query never executes and the query is where the bug is.
- A repository class that only forwards to the ORM. That is a layer with no behavior.
- Building the UI and the data layer as separate sessions. The slice is one session because neither half is verifiable alone.
- `ON DELETE SET NULL` on a nullable foreign key. It orphans children silently. Use `restrict` unless there is a stated reason.
- A trigger or guard scoped to one operation type. A rule that fires on update is absent during delete-then-insert, which is what an importer does.

## Gate

```
pnpm db:migrate && pnpm db:migrate:down && pnpm db:migrate
pnpm db:seed && pnpm gate
```

Adversarial check: import from `src/server/` into a client component, expect a build error. Run the seed against the wrong host, expect a refusal naming it. Delete a parent row and confirm nothing referencing it was silently orphaned.
