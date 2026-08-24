# Phase 2: Authentication

**Skills:** `auth-flows`, `tdd-workflow`, `api-and-validation`
**Gate:** `docs/phases.md`, Phase 2

Read `.claude/skills/auth-flows/references/flow-trees.md` first. Every branch there is built or explicitly declined in `docs/product.md`.

## Build

**Clients.** Three, kept distinct and never mixed: server (reads cookies), browser (in-memory session), middleware (refreshes tokens). Put them in `src/lib/auth/` with names that make a wrong import obvious.

**Flows.** Signup, email verification, login, logout, password reset, session refresh. Every branch in the trees.

**`src/server/auth.ts`.** `requireUser()`, `requireOwner()`, session helpers. Marked `import "server-only"`.

**Middleware.** Token refresh plus optimistic redirect for protected paths. Not a security boundary and the code should say so in a comment.

**Rate limits.** Login, reset request, verification resend.

**RLS.** Enabled on every table, default deny, policies for the user-owned tables that exist so far.

**Redirect to intent.** After login the user lands where they were going. This is the branch that silently regresses. Test it explicitly.

## Avoid

- Any enumeration signal. Signup, login, and reset return the same message whether the account exists or not.
- A token that survives use. Consumed means dead, and a second use routes to login rather than erroring.
- Auth checks in middleware only. The real check is at data access.
- A verification email whose link lands on the provider's page. Set the redirect target to the app.
- Session state in localStorage.
- Building the happy path and deferring the branches. The branches are the phase.

## Gate

```
pnpm gate
pnpm test:e2e -- auth
pnpm test -- authz
```

The authorization test is the one that matters: authenticate as user A, request user B's resource by id, on every parameterized route, expect 404. If it passes trivially, confirm the route exists before believing it.
