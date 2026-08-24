---
name: api-and-validation
description: "Write or review route handlers, server actions, and the data access layer. Use when adding an endpoint or a mutation, validating input, shaping an error response, adding pagination, handling idempotency, deciding what data crosses to the client, or adding rate limiting. Triggers include 'route handler', 'server action', 'API', 'endpoint', 'mutation', 'validate', 'DTO', 'pagination', 'rate limit', 'this returns too much data'. Enforces parse-at-the-boundary, authorization inside the query, and DTOs out. Do NOT use for database schema or query optimization; that is db-and-migrations."
---

# API and validation

Every server action and route handler is a public POST endpoint. Nothing about being called from your own UI makes it safe.

## The shape

Transports are thin. Three steps, no logic:

```
1. Parse input with a schema. Unknown shape in, typed value out.
2. Call src/server/. That layer authenticates, authorizes, and queries.
3. Shape the response. Return a DTO or a Result.
```

If a transport has a branch that is not about input or output shape, that branch belongs in `src/server/` or `src/core/`.

## Parse at the boundary

Schema-validate everything arriving from outside: request bodies, search params, route params, form data, webhook payloads, third-party responses, environment. Inside the boundary the types are real. Outside, nothing is.

A passing schema is not authorization. A well-formed object can still point at a row the caller does not own. These are two separate checks and both are required.

## The data access layer

```ts
import 'server-only';

export async function getOrderForUser(orderId: string): Promise<OrderDTO | null> {
  const user = await requireUser();
  const row = await db.query.orders.findFirst({
    where: and(eq(orders.id, orderId), eq(orders.userId, user.id)),
    columns: { id: true, status: true, totalCents: true },
  });
  return row ? toOrderDTO(row) : null;
}
```

Four things that all matter: `server-only` so a client import is a build error, the auth check inside the function, the ownership predicate inside the same query rather than an if afterward, and explicit columns.

Never fetch your own API from your own server. Call the function.

## DTOs

The client receives a shape designed for the view, never a database row. A row carries columns nobody meant to expose, and the leak is silent because it renders fine.

Every DTO is a deliberate list of fields. When a new column is added, nothing new reaches the client until someone decides it should.

## Errors out

| Situation | Status | Body |
|---|---|---|
| Validation failed | 400 | Field-level messages, no internals |
| Not authenticated | 401 | Generic |
| Not authorized or not found | 404 | Identical for both. A 403 confirms the resource exists |
| Conflict | 409 | What conflicted, in user terms |
| Rate limited | 429 | Retry-after |
| Upstream failed | 502 or 503 | Whether retrying helps |
| Unexpected | 500 | Generic message plus the request id. The id is what makes support possible |

Returning 404 rather than 403 for another user's resource is deliberate. A 403 tells an attacker the id was real.

## Idempotency

Any endpoint that creates something a user could double-submit accepts an idempotency key and returns the original result on replay. Checkout, order creation, invite sending.

The key is stored with the result and constrained unique. Not checked with an if, which loses the race that the feature exists to prevent.

## Pagination

Cursor-based for anything that grows. Offset pagination drifts when rows are inserted mid-scroll and gets slow at depth. Return the cursor, never a total count on a large table unless someone asked and accepted the cost.

## Rate limiting

Every mutation and every auth endpoint. Keyed by user when authenticated, by IP when not. The limit is a named constant in one place, not scattered magic numbers.

## Review checks

- Is every input parsed, including route params?
- Is ownership checked inside the query?
- Does the response contain only fields someone chose to expose?
- Does the transport contain logic that belongs a layer down?
- Can this be double-submitted, and does that matter?
