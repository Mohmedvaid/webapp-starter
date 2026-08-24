# Phase 5: Hardening

**Skills:** `security-review`, `preflight-and-observability`, `api-and-validation`
**Gate:** `docs/phases.md`, Phase 5

Read `.claude/skills/security-review/references/checklist.md` and run the whole pass.

## Build

**Request correlation.** A request id generated in middleware, attached to every log line, returned in the error response body. Without it, support is guesswork.

**Error taxonomy applied.** Every transport returns from the `AppError` union. No raw error, stack trace, or internal name reaches a user. 404 rather than 403 for another user's resource.

**Rate limits.** Every mutation and every auth endpoint. Limits as named constants in one module.

**Headers.** CSP without `unsafe-inline`, HSTS, `frame-ancestors` none, nosniff, referrer policy. Cookies HTTP-only, secure, SameSite.

**Error monitoring.** Wired, with a dedicated alert on webhook handler failure, separate from the general error rate. At low volume the general rate will not surface it, and a silent webhook failure is a customer who paid and got nothing.

**Semgrep.** Added to `pnpm gate` alongside `pnpm audit`. CI still runs `pnpm gate` and nothing else.

**Authorization matrix.** The table in `docs/architecture.md` filled and verified row by row against the actual code. A matrix that has drifted is worse than no matrix.

## Avoid

- Logging the traversal instead of the decision. "Entering getEntitlement" is noise. "Entitlement denied, expired 2 days ago" is a log.
- Redaction implemented at call sites. It is a default in the logger or it will be forgotten.
- A 403 where a 404 belongs. A 403 confirms the id was real.
- Building a dashboard before the logs are correlated. Rung 3 before rung 0.
- Adding rate limiting as a wrapper around every handler individually. One middleware, one config.

## Gate

```
pnpm gate
pnpm test:e2e -- rate-limit
```

Then: trigger one handled error and one unhandled error. Confirm the user sees something useful, the log carries the request id, and the monitor fired.
