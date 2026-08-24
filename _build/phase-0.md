# Phase 0: Foundation

**Skills:** `preflight-and-observability`, `tdd-workflow`, `git-workflow`
**Gate:** `docs/phases.md`, Phase 0

The app boots, refuses to boot when misconfigured, and CI is green before a single feature exists.

## Build

**Scaffold.** Next.js App Router, TypeScript strict, pnpm, Node 22 pinned in `.nvmrc` and `engines`. Tailwind. Vitest. Playwright.

**`src/lib/env.ts`.** Zod schema, separate client and server objects, parsed at module import so a bad config stops the boot. Export the typed object. This is the only file in the repo that reads `process.env`.

**`src/lib/preflight.ts`.** Runs before traffic. Checks: database reachable, migrations current, payment key prefix matches `NODE_ENV`, webhook secret present, sending domain set. Each failure names the specific check. Exposed as `pnpm preflight`.

**`src/app/api/health/route.ts`.** Process alive. No dependency checks. Always 200 if the process responds.

**`src/app/api/ready/route.ts`.** Database plus migration state. 503 with the failing dependency named in the body.

**`src/lib/result.ts`, `src/lib/errors.ts`.** The `Result` type and the `AppError` union from the `preflight-and-observability` skill.

**`src/lib/logger.ts`.** Structured JSON, level from env, redaction by default for tokens, keys, passwords, emails, payment fields.

**`middleware.ts`.** Security headers, request id generation, rate limit hook. No auth logic. Middleware is an optimistic redirect and nothing more.

**`.env.example`.** Matches the schema exactly, every variable, dummy values.

**Lint.** ESLint with: the import-boundary rule enforcing `app` to `features` to `server` to `core`, a restricted-syntax rule banning `process.env` outside `src/lib/env.ts`, and a rule banning literal hex colors and raw px in `src/components` and `src/features`. Prettier. commitlint. gitleaks as a pre-commit hook and in the gate. knip for dead exports.

**`pnpm db:drift`.** `scripts/drift.ts`. Compares applied migrations against committed ones and fails **both** directions. Generic comparison logic, one small adapter per migration tool, selected at `project-init`. Exits non-zero with a named message when credentials are absent. Never skips.

**`pnpm gate`.** One command: typecheck, lint, knip, unit, e2e, production build, `db:drift`, `pnpm audit`, gitleaks. This is the gate. See `docs/adr/0002`.

**CI.** `.github/workflows/ci.yml` runs `pnpm gate` and nothing else. No parallel step list.

## Avoid

- A config module that reads env lazily. Parse at import or the failure moves to runtime, which is the bug this phase exists to prevent.
- Health and ready as one endpoint. They answer different questions and merging them causes restart loops.
- A generic "preflight failed" message. Name the check.
- Wrapping the logger in a second logger. Use the library directly, in one module.
- Adding a dependency not needed for this phase.
- A drift check or preflight check that returns success when it cannot reach its dependency. Refuse.
- Writing a check without a test that proves it fires. A guard nobody calls passes its unit test and protects nothing.

## Gate

```
pnpm install && pnpm preflight && pnpm dev
curl localhost:3000/api/ready
pnpm gate
```

Adversarial check, both parts pasted: delete a required variable and confirm the app refuses to start and names it. Then unset the drift check's credentials and confirm it exits non-zero rather than passing.
