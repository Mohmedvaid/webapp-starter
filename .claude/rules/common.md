# Common rules

Always loaded. Only rules that change generated code on nearly every task belong here.
If a rule applies to one domain, it belongs in that skill instead.

## Correctness

- Parse external input at the boundary with a schema. Inside the boundary, types are real. Outside, nothing is.
- Expected failures are returned as values. Unexpected failures are thrown and alerted on. If the UI renders it differently, it is a return value.
- No silent catches. A caught error is logged with context or rethrown. Never both swallowed and ignored.
- Every external call has an explicit timeout. A hanging call is worse than a failing one.
- Retries need backoff and a ceiling. Unbounded retry is an outage amplifier.
- A check that cannot run must refuse, never skip. A skipped check reports success it did not earn.
- A guard that is never called is worse than absent, because the test passes and the risk is filed as handled.

## State and data

- Record transitions as timestamps, not booleans. `approved_at` beats `is_approved` because it debugs itself.
- More than two outcomes means an enum, not a boolean. Add the enum on day one.
- The database enforces invariants that matter. Unique constraints and foreign keys, not application checks alone.
- Anything that can be delivered twice must be idempotent. Webhooks, queue consumers, retried jobs.
- Explicit column selection. Never `select *` into something a user will see.
- Scope a guard to the invariant, not to an operation type. A rule that only fires on update does not survive a delete-then-insert.
- Omissions are asymmetric. A missing addition breaks loudly on first use. A missing revocation, constraint, or policy breaks nothing and leaves the unsafe state live.
- Bulk writes upsert by key. Delete-and-reinsert destroys anything referencing those rows, silently when the reference is nullable.

## Security

- Authorization is checked where data is accessed, not where the route is defined.
- Authentication is not authorization. Confirm the caller owns the resource, not just that they are logged in.
- Error messages never reveal whether an account exists, which field failed, or what the internals are named.
- Secrets are read in exactly one module and never logged. Log lines are redacted by default, not by remembering.
- Tokens are single-use and expiring: verification, reset, invite, magic link.

## Configuration

- Zero hardcoded values that differ between environments. URLs, keys, limits, feature flags.
- Config is validated once at startup and fails the boot loudly. A misconfigured app must not accept traffic.
- No literal colors, fonts, spacing, radii, or z-index values in components. Tokens only.

## Structure

- Business logic is pure and framework-free. It takes data and returns data. That is the layer worth unit testing.
- Imports go one direction. Peers do not import peers. Shared code moves down, never sideways.
- Tests sit next to the file they test.
- One responsibility per module. If the filename needs "and", split it.
- Nondeterminism is injected, never reached for. Clock, randomness, and id generation are parameters, which is what makes the logic testable.

## Working method

- Reproduce before fixing. A bug without a failing test is a guess.
- Every bug fix ships with the regression test that would have caught it.
- Read the existing pattern before adding a new one. Consistency beats personal preference.
- When two approaches are close, pick the one that is easier to delete.
- State disagreement once with the reason, then do it the way you were asked.
- Local green does not mean production correct. A local chain that resets and reseeds guarantees local is current, which is exactly why it cannot see drift, stale data, or provider settings.

## Honesty

- Never report a check as passing without running it and reading the output.
- Never describe work as complete when a test is skipped, a type is suppressed, or a case is unhandled.
- Unknowns are stated as unknowns. A confident wrong answer costs more than a question.
