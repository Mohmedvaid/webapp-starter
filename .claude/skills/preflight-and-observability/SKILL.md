---
name: preflight-and-observability
description: "Configuration, startup validation, health checks, logging, and error handling. Use when adding or changing an environment variable, when the app will not boot, when adding a health or readiness endpoint, when setting up logging or error monitoring, when deciding how errors are shaped and surfaced, or when diagnosing a failure that only happens in one environment. Triggers include 'env var', 'it won't start', 'health check', 'readiness', 'logging', 'error handling', 'works locally but not in production', 'restart loop'. Covers the small infrastructure that quietly decides whether failures are loud or silent."
---

# Preflight and observability

The category of bug this prevents: an app that starts successfully with a broken configuration and fails at checkout three hours later.

## Environment

One schema, one module, parsed at import.

```
src/lib/env.ts    zod schema, split client and server, parsed on import
```

Rules:
- `process.env` is read here and nowhere else. Enforced by lint.
- Parsing happens at import, so a bad config stops the boot rather than surfacing at runtime.
- Server-only variables live in a schema that is never imported by client code.
- `.env.example` matches the schema exactly. A variable in one and not the other is a gate failure.
- Every variable also appears in the table in `operations.md` with its owner.
- Defaults are for things that genuinely have a safe default. `PORT` yes. `DATABASE_URL` never.
- Dangerous defaults point the safe way. Indexing defaults to noindex when unset, so a half-built site cannot get crawled by omission.

## Preflight

Runs before the app accepts traffic. Checks the things that are valid individually but wrong together:

- Database reachable
- Migrations current, not pending
- Payment key prefix matches `NODE_ENV`. A live key in development is a real charge on a real card
- Webhook secret present
- Sending domain configured
- Required buckets or external resources exist

Failure exits with a message naming the specific check. "Preflight failed" is not a message.

Two properties, both learned the hard way:

- **Refuse, never skip.** A check that cannot reach its dependency exits non-zero. Skipping turns an unverifiable state into a reported success.
- **Every check has a test that proves it fires**, not only that its logic is right. A guard written and never wired passes its unit test and protects nothing.

## Provider settings

Some of your configuration lives in someone else's dashboard: token and code lengths, email template bodies, redirect URL allowlists, webhook endpoints, plan limits, sending domain verification. The repo cannot see any of it, and a provider default is not your default.

Every provider setting your code depends on gets a row in the table in `operations.md`, and where the provider exposes it, a preflight assertion. The ones that bite: a code length that does not match what the UI accepts, a redirect URL registered for one environment and not another, and a sending domain that is unverified so mail silently vanishes.

## Health versus readiness

Two endpoints because they answer different questions, and conflating them causes restart loops where a database blip kills a healthy process.

| Endpoint | Question | Checks | Failure means |
|---|---|---|---|
| `/api/health` | Is the process alive | Nothing external | Restart the process |
| `/api/ready` | Can it serve traffic | Database, migration state | Stop routing traffic here |

`/api/ready` names the failing dependency in its body. A 503 with no detail wastes the first five minutes of every incident.

## Logging

- Structured JSON, never a bare console call in production paths.
- A request id generated in middleware and attached to every line for that request. Without correlation, logs are noise.
- Redaction is a default in the logger, not something remembered at each call site. Tokens, passwords, keys, full email addresses, payment details.
- Log level from the environment. `info` in production.
- Log the decision, not the traversal. "Entitlement denied, expired 2 days ago" is useful. "Entering getEntitlement" is not.

## Errors

Expected failures are returned. Unexpected ones are thrown and alerted.

```
type AppError =
  | { kind: 'not_found' }
  | { kind: 'forbidden' }
  | { kind: 'validation'; fields: Record<string, string> }
  | { kind: 'conflict'; reason: string }
  | { kind: 'upstream'; service: string; retryable: boolean }
```

Rule of thumb: if the UI renders it differently, it is a returned value. If it means the code is wrong, it is a throw plus an alert.

Users never see a raw error, a stack trace, or an internal name. They see what failed, whether retrying helps, and what to do next.

## Alerts

One alert is non-negotiable and separate from the general error rate: **webhook handler failure**. A silent webhook failure is a customer who paid and got nothing, and the general error rate will not surface it at low volume.

## Environment-only failures

When something works locally and fails deployed, check these before anything clever: cookie domain, absolute app URL, redirect URLs registered with providers, sending domain verification, key mode, and whether the variable is actually set in the platform rather than only in a local env file.
