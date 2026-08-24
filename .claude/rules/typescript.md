# TypeScript and Next.js rules

Always loaded. Stack-specific counterpart to `common.md`.

## TypeScript

- `strict` is on and stays on. No `any`. No non-null assertion without a comment justifying it in one line.
- Prefer `unknown` plus a schema parse over a cast. `as` is a claim you are making without evidence.
- Discriminated unions over optional-field soup. If two shapes are different, model them as different.
- `satisfies` for config objects so you keep inference and still get checked.
- Types describe data, not classes. Interfaces for extensible contracts, type aliases for everything else.
- Exported functions have explicit return types. Inference is fine internally, not at a module boundary.
- No enums. Use `as const` objects and derive the union.
- Errors are typed. A catch block receives `unknown` and narrows before use.

## Next.js App Router

- Server Components by default. `"use client"` only where interactivity or browser APIs demand it, and as deep in the tree as possible.
- Middleware performs optimistic redirects only. It is not a security boundary. Post CVE-2025-29927 this is the framework's own position.
- Every Server Action is a public POST endpoint. Validate arguments, re-authenticate, and check ownership inside the action or the layer it calls.
- Data access lives in `src/server/`, marked `import "server-only"`. Client components cannot import it. That build error is the point.
- Return DTOs shaped for the view. Never pass a full database row through a component boundary.
- Never `fetch` your own API from your own server. Call the function directly.
- Route handlers that receive webhooks read the raw body. No parser in front of a signature check.
- `after()` for post-response work that must not delay the response. Anything longer than the function duration cap belongs in a job runner, not a request.

## React

- Derive state, do not duplicate it. If it can be computed from props, compute it.
- `useEffect` is for synchronizing with something outside React. Not for transforming data, not for fetching in a server-rendered app.
- Keys are stable identifiers from the data. Never the array index for anything reorderable.
- Every async surface handles four states: loading, empty, error, success. Missing empty state is the usual gap.
- Forms use Server Actions with progressive enhancement where possible. Client-side validation mirrors server-side validation and never replaces it.

## Styling

- Tailwind utilities consuming CSS variables from `src/styles/tokens.css`. No arbitrary values carrying raw colors or sizes.
- Component variants through `cva` or an equivalent, not through prop-driven string concatenation.
- Dark mode via token swap at the root, never per-component conditionals.

## Data layer

- Drizzle schema is the single source of truth. Types derive from it.
- Every migration is reversible and reviewed before it runs anywhere real.
- Transactions wrap multi-write operations that must not partially apply.
- Serverless plus Postgres requires a connection pooler. Assume it, configure it, note it in `operations.md`.

## Testing

- Vitest for unit and integration. Playwright for end to end.
- Pure logic in `core/` is unit tested directly. No mocks needed, that is why it is pure.
- Data access is tested against a real database, seeded and torn down. Mocking the database tests the mock.
- End to end covers the money paths and the auth paths. Not every screen.
- No test asserts on implementation details. Assert on the behavior a user or caller observes.

## Tooling

- pnpm. Node 22 LTS. Version pinned in `.nvmrc` and `package.json` engines.
- ESLint with the import-boundary rule enforcing the layer directions in `CLAUDE.md`.
- Conventional commits. The type prefix is not decoration, it drives the changelog.
