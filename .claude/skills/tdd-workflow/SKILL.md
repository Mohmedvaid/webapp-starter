---
name: tdd-workflow
description: "The required workflow for every implementation task and every bug fix in this repo. Use whenever code is about to be written or changed: a new feature, a bug fix, a refactor, or a change to existing behavior. Triggers include 'implement', 'build', 'add', 'fix', 'this is broken', 'write the code for', or any task handed over by phase-planning. Enforces failing-test-first with captured evidence, and the rule that every bug ships with the regression test that would have caught it. Do NOT use for deciding what to test at what level; that is testing-strategy."
---

# TDD workflow

Not a suggestion in this repo. The evidence trail is the deliverable, not just the code.

## The cycle

**RED.** Write the test. Run it. Capture the failure output. A test that has never failed proves nothing, because it may be asserting something that was already true.

**GREEN.** Write the least code that passes. Not the design you have in mind, the least code. The design comes in the next step, informed by a passing test.

**REFACTOR.** Clean it up with the test still green. This is where the abstraction decision happens, and by then you know whether it is needed.

Paste the RED output before the implementation. If a session claims a test was failing without showing it, that claim is unverified and the work is not done.

## Bugs

1. Reproduce it with a failing test first. A fix without a failing test is a guess that happened to change the symptom.
2. Fix it.
3. The test stays. That is the regression test, permanently.

Every bug fix ships with the test that would have caught it. No exceptions, and this is the single highest-value rule in the repo.

## What gets tested first

Pure logic in `core/` and the feature's `core/`. It needs no mocks, which is the entire reason the layer is pure. Test it directly and thoroughly.

Then data access against a real seeded database. Mocking your own database tests the mock.

Then the transport, only for its own concerns: input parsing, status codes, redirect behavior. Not the logic underneath, which is already covered.

## Never

- Never delete or skip a failing test to go green. A skipped test is a lie with a green checkmark.
- Never weaken an assertion to make it pass. If the assertion was wrong, say so and explain why the new one is right.
- Never write the implementation first and the test after. The test written afterward tests what the code does, not what it should do, and it will pass for the wrong reason.
- Never claim a suite passes without running it and reading the output.

## When TDD genuinely does not fit

Exploratory spikes and UI layout iteration. Both are allowed, both are throwaway.

The rule: a spike is deleted, not promoted. If a spike becomes the implementation, it gets written again with tests first. Saying "the spike already works" is how untested code enters a codebase permanently.

## Gate

```
pnpm test            # green, zero skipped
pnpm typecheck       # clean
pnpm lint            # clean
```

Zero skipped is part of the gate. A skipped test is either a missing feature that belongs in the backlog or a broken test that belongs fixed.
