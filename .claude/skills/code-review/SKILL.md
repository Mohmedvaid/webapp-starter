---
name: code-review
description: "Review code before merge from a fresh context. Use when a builder session has finished and the work needs an adversarial pass, when Mohmed asks 'review this', 'check my work', 'is this ready to merge', or when a pull request is open. This is the reviewer half of the one-builder-then-one-reviewer rule and runs in a separate session from the one that wrote the code. Orders findings by cost of being wrong and names a mechanism for every finding. Do NOT use for a security-specific pass before deploy; that is security-review, which is deeper and narrower."
---

# Code review

Runs in a session that did not write the code. That is the whole point: the context that wrote it cannot see what it assumed.

## Order findings by cost of being wrong

Not by how easy they are to spot. Stop at the first three if present.

1. **Silent data or money loss.** Failure that surfaces as success.
2. **Authorization gaps.** Ownership not checked inside the query.
3. **Correctness under retry or concurrency.** Duplicate delivery, two tabs, races.
4. **Unhandled failure modes.** Missing states, no timeout, third-party outage.
5. **Irreversibility.** Destructive migration, deletion with no recovery.
6. **Coupling.** Feature importing a feature, logic in a transport, vendor leaking into the domain.
7. **Slop.** Wrappers, one-implementation interfaces, defensive checks for impossible states, comments restating code.
8. **Style.** Last, and only where it is inconsistent with the repo rather than with taste.

## Questions to run

**State.** What happens if this runs twice? Twice at once? What is recorded versus inferred? What is the state after a partial failure?

**Failure.** Is the third party hanging or erroring? Are retryable and non-retryable failures distinguished? What does the user see, what does the log say, who gets alerted?

**Authorization.** Where is ownership checked, and is it in the same query? What if the id belongs to another user?

**Boundary.** What crosses to the client? Does any DTO carry a field nobody chose to expose? Is `process.env` read outside its one module?

**Call sites.** What calls this? Does a test go through that path, or only through the function directly? A new export with no importer is either unfinished or a deleted call site.

**Tests.** Did the test ever fail? Does it assert an outcome or a mock? Is anything skipped?

## Finding shape

Three parts, always:

1. **Mechanism.** What goes wrong, in what sequence. Not "this could be a problem."
2. **Trigger.** What has to happen. If it is already broken, say that first.
3. **Fix, sized.** One line, refactor, or redesign.

A finding without a mechanism is a vibe. Without a trigger it cannot be prioritized. Without a size it cannot be scheduled.

## Verify rather than trust

- Run the gate commands yourself. Do not accept a claim that they passed.
- Read the test, not just its name. A test named `rejects unauthorized access` that asserts a 200 has shipped before.
- Check the diff for things not mentioned in the task. Unrequested adjacent refactors are how a small change becomes a large risk.

## What not to do

- Do not list every possible improvement. Cut where the cost stops justifying attention.
- Do not soften a real finding to be agreeable, or inflate a small one to appear thorough.
- Do not re-open decisions already recorded in an ADR unless new information invalidates them, and then name the information.
- Do not review the person. Review the artifact.

## Output

A short verdict, then the ordered findings, then the gate output. If it is ready to merge, say so plainly. If it is not, name the smallest set of changes that would make it ready, rather than everything that could theoretically improve.
