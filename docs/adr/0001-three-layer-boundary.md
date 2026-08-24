# ADR 0001: Three-layer boundary between core, server, and transport

**Status:** Accepted
**Date:** <!-- FILL -->
**Supersedes:** none

> This file is also the format example. Copy it for every subsequent ADR. Keep them short. An ADR nobody reads is a decision nobody remembers making.

## Context

A web app in this stack can put logic in four places: components, route handlers, server actions, or a service layer. Left unconstrained, it ends up in all four, and the same rule gets implemented three times with two of them subtly wrong.

Two forces push on the decision:

1. **Testability.** Logic that touches a request, a session, or a database is expensive to test and gets tested badly or not at all.
2. **Optionality on the backend shape.** Work that outgrows a request cycle needs a job runner or a worker. If that work is entangled with HTTP, moving it is a rewrite rather than a new entry file.

## Decision

Three layers, strict one-way dependency:

```
transports  →  src/server/  →  src/core/
```

- `src/core/` imports no framework, no database client, no fetch. Pure functions. Data in, data out.
- `src/server/` owns data access and authorization. Every function authenticates, verifies ownership inside the query, selects explicitly, and returns a DTO. Marked `import "server-only"`.
- Transports (route handlers, server actions, webhook routes, cron entries, and any future worker) are thin. Parse input, call `src/server/`, shape the response. No logic.

Enforced by an ESLint import-boundary rule, not by convention.

## Consequences

**Good**
- The layer worth unit testing is pure, so it is tested without mocks. Mocking a database tests the mock.
- Authorization has one home. Auditing it means reading one directory.
- Adding a worker or moving a job off the request cycle is additive. Nothing below the transport layer knows what invoked it.
- A client component importing server code is a build error, not a production leak.

**Bad**
- More files for a trivial feature. A three-field CRUD form touches three layers.
- Real discipline required on the "no logic in transports" rule. It erodes first under deadline pressure, which is why it is lint-enforced rather than documented.
- Some duplication between a `core` type and its DTO. Accepted, because collapsing them is what leaks database columns to clients.

**Neutral**
- Does not commit to a separate API service, and does not preclude one. It makes extraction cheap if the conditions ever justify it.

## Alternatives considered

| Alternative | Rejected because |
|---|---|
| Logic directly in Server Components | Framework's own guidance calls this prototyping-only. No single place to audit authorization |
| Full hexagonal architecture with ports and adapters | Boilerplate cost exceeds the return at this size. The three-layer split captures most of the benefit |
| Separate API service from day one | Solves a problem this project does not have, at permanent operational cost. See the escalation ladder in `architecture.md` |

## Revisit when

The transport layer stops being thin, or a second first-class client appears. Either is a signal to re-open, not to quietly work around.
