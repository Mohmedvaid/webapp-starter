---
name: auth-flows
description: "Build or review authentication and session handling. Use for anything touching signup, email verification, login, logout, password reset, magic links, OAuth or social login, MFA, session refresh, account lockout, email change, account deletion, or protecting a route. Triggers include 'login', 'sign up', 'auth', 'session', 'password reset', 'user is null', 'random logouts', 'protect this page', 'who can access this'. Load references/flow-trees.md for the full branch trees before implementing any flow. Do NOT use for deciding what a user is allowed to do once identified; ownership and entitlement checks belong to api-and-validation and security-review."
---

# Auth flows

The happy path is the easy ten percent. This skill is about the other ninety.

Load `references/flow-trees.md` before implementing any flow. Every branch in those trees is built or explicitly declined, never silently skipped.

## Invariants

1. **Never leak account existence.** Signup, login, and reset all return the same message whether the account exists or not. "If an account exists, we sent a link."
2. **Every token is single-use and expiring.** Verification, reset, invite, magic link. Consumed means dead, and a second use routes to login rather than erroring.
3. **Authorization is checked at data access, not in middleware.** Middleware is an optimistic redirect. It is not a security boundary, and the framework's own guidance says so.
4. **Redirect to intent.** After login, the user lands where they were trying to go. This is the branch that silently regresses most often. Test it.
5. **Rate limit every auth endpoint.** Login, reset request, and verification resend especially. A reset endpoint without a limit is a mail-bomb service.

## The three-client trap

The server client reads cookies, the browser client holds an in-memory session, the middleware client refreshes tokens. Mixing them produces "user is null" bugs that look like broken auth but are import mistakes.

Symptoms and causes:

| Symptom | Cause |
|---|---|
| `user is null` in a server component | Browser client imported on the server |
| Random logouts after navigation | Middleware not refreshing, server sees a stale session |
| Session works locally, fails in production | Cookie domain or redirect URL mismatch between environments |
| Verification link lands on the provider's page | Missing redirect-to-app parameter on signup |

## Sessions

- HTTP-only cookies. Nothing auth-related in localStorage.
- Access token refresh happens in middleware. Skip it and server components see expired sessions.
- Decide and write down, in `architecture.md`: does a password reset revoke other sessions, and does account deletion revoke immediately or on next request.

## OAuth

The collision branch is mandatory: the email is already registered with a password. Offer linking after re-authentication. Never silently duplicate the account, and never silently merge them.

Also handle: provider returns without an email, user denies consent, provider outage, permissions revoked between visits.

## Unverified accounts

Decide explicitly what an unverified user can do and for how long, then write it in `product.md`. Leaving this undecided means it gets decided accidentally by whichever route forgot to check.

## Gate

```
pnpm test:e2e -- auth      # every branch in flow-trees.md, positive and negative
pnpm test -- authz         # cross-user access returns 404
```

The cross-user test is the one that matters: authenticate as user A, request user B's resource by id, on every parameterized route. If it passes trivially, confirm the route actually exists before believing it.
