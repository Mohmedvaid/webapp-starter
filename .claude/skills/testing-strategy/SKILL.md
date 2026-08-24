---
name: testing-strategy
description: "Decide what to test, at what level, and how. Use when asking what tests a change needs, when setting up test infrastructure, factories, or fixtures, when a test is slow, flaky, or hard to write, when deciding whether something needs an end-to-end test, or when discussing coverage. Triggers include 'what should I test', 'write tests for', 'coverage', 'e2e', 'this test is flaky', 'how do I test this', 'do I need a test for'. Do NOT use for the red-green-refactor cycle itself or the failing-test-first rule; that is tdd-workflow. This skill decides WHAT and WHERE, tdd-workflow decides the ORDER."
---

# Testing strategy

## The split

| Level | Tool | Covers | Speed |
|---|---|---|---|
| Unit | Vitest | Pure logic in `core/` | Milliseconds |
| Integration | Vitest plus real database | `src/server/`, queries, authorization | Seconds |
| End to end | Playwright | Money paths and auth paths only | Slow |

Most tests should be unit tests of pure logic, which is the entire reason `core/` is framework-free. A layer that needs mocks to test is a layer that was built wrong.

## What gets which

**Unit.** Pricing, scheduling, grading, validation rules, state machines, formatting, anything that takes data and returns data. No mocks needed, so no mocks used.

**Integration.** Anything touching the database. Run against a real seeded database. Mocking your own data layer tests the mock, and the query, which is where the bug actually is, never executes.

**End to end.** Signup through first value. Purchase through entitlement. Login, reset, logout. That is close to the whole list. Not every screen, because e2e is expensive and slow and each one is a maintenance obligation.

## Call sites

Testing a function cannot see a function nobody calls. Two real failures of this shape: a guard that would have refused a live payment key in development, written and never invoked, and a structured-data builder that was unit tested and rendered nowhere. Both had passing tests. Both did nothing.

- A guard's test asserts it **fires in the real path**, not that its predicate is correct in isolation.
- A builder or formatter gets one test that renders it through the surface that is supposed to mount it.
- `pnpm gate` includes a dead-export check. An export nothing imports is either unfinished work or a deleted call site, and both are worth failing on.

The question in review is not "is this function correct" but "what calls this, and does a test go through that path."

## What local tests cannot catch

Naming these so nobody reads green as correct:

| Invisible locally | Why | Caught by |
|---|---|---|
| Schema drift | The local chain resets and reseeds, so local is always current | `pnpm db:drift`, both directions |
| Data state | Production content can be a fraction of what git holds | Post-deploy data assertion |
| Provider settings | Token lengths, template contents, redirect URLs, plan limits live in a dashboard, not the repo | Preflight assertions plus the operations table |
| Environment config | Cookie domains, key modes, sending domains | Production smoke set |

Everything in that table has shipped a real outage. None of them are test failures.

## Test doubles

Mock things you do not own and cannot run: a payment provider, an email service, a third-party API.

Do not mock things you own: your database, your own modules, your own functions. If a module is hard to test without mocking your own code, the coupling is the finding.

Never assert that a mock was called. That tests the test. Assert on the observable outcome.

## Factories

Every entity has a factory producing a valid instance with overridable fields. Tests that build objects by hand break every time a required column is added, and the resulting churn teaches people to avoid writing tests.

Seed data is deterministic: known ids, one user in each meaningful state, at least one row of every entity.

## Flaky tests

A flaky test is a bug, either in the test or in the code, and treating it as noise is how real race conditions get shipped.

Common causes in order: waiting on a timeout instead of a condition, shared state between tests, real time instead of a controlled clock, test-order dependence, and unawaited promises.

Never retry a flaky test to make CI green. Fix it or delete it and record the gap.

## Coverage

CI reports the number and uploads the report. There is no threshold gate until Mohmed sets one after MVP, when he can see what is actually covered.

The rule that does the real work: **every bug ships with the regression test that would have caught it.** Coverage measured that way rises where it matters, rather than where it is easy.

## What not to test

- Framework behavior. It is already tested.
- Getters, constructors, and pass-through glue.
- Exact copy strings, unless the string is a legal requirement.
- Implementation details. If a refactor with unchanged behavior breaks a test, that test was coupled to structure.

## Gate

```
pnpm gate              # the whole chain, zero skipped
```

Zero skipped is part of the gate. A skipped test is either a missing feature belonging in the backlog or a broken test belonging fixed.
