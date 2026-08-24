# ADR 0002: One gate command, CI calls it

**Status:** Accepted
**Date:** <!-- FILL -->
**Supersedes:** none

## Context

A solo project needs to decide what stands between a change and production. Two positions are defensible and they are usually posed as a fork.

**CI as the gate.** Checks run on a neutral machine on every push. Catches what does not reproduce locally: a missing lockfile entry, a machine-specific path, a stale branch. Costs setup, minutes, and a second list of checks that drifts from the local one.

**Human merge as the gate.** No CI, no branch protection. The owner runs checks locally and merges. Fast, free, and honest about the fact that a solo builder gains nothing from an approval gate only he can satisfy. Costs everything that only shows up on another machine.

The fork is false. What actually goes wrong is not "we chose the wrong one," it is that the two lists diverge, and then a green tick in one place means something different from a green tick in the other.

A related failure motivated this: migrations and code shipping on separate tracks, with local always current because the local chain resets and reseeds. Local was green for six days while production was four migrations behind, two of them authorization fixes. No arrangement of gates helps if the gate cannot see the thing that is wrong.

## Decision

**One command, `pnpm gate`.** Typecheck, lint, dead-export check, unit, integration, e2e, production build, Lighthouse budgets, migration drift, dependency audit, static analysis, secret scan.

CI runs `pnpm gate` and nothing else. No parallel list of steps in a workflow file.

The gate includes checks that reach outside the repo, specifically migration drift, and those refuse rather than skip when they cannot reach their dependency.

Mohmed's merge remains the only human gate. Branch protection stays off.

## Consequences

**Good**
- Local green and CI green mean the same thing, by construction rather than by discipline.
- Adding a check is one edit in one place.
- The gate can see production state, so drift and data problems are catchable rather than structurally invisible.
- A project that later drops CI loses redundancy, not coverage.

**Bad**
- The gate is slow. E2E and a production build are not seconds. Fast subcommands exist for the inner loop, but the gate is the gate.
- A check that reaches outside the repo needs credentials in more places, and needs to refuse loudly when they are absent. That refusal will occasionally be annoying and must not be softened into a skip.

**Neutral**
- CI becomes near-trivial, which is the point. It is redundancy against a forgotten local run, not a separate authority.

## Alternatives considered

| Alternative | Rejected because |
|---|---|
| CI with its own step list | The two lists drift and green stops meaning one thing |
| Local only, no CI | Nothing catches a forgotten run or a stale branch, and the cost of the workflow file is near zero once it just calls one command |
| Branch protection requiring CI | A solo builder blocking himself behind a check only he can satisfy adds ceremony, not safety |

## Revisit when

A second person commits to the repo. At that point branch protection stops being ceremony and starts being a real control, and this decision should be reopened rather than worked around.
