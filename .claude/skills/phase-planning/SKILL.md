---
name: phase-planning
description: "Break a phase from docs/phases.md into tasks and builder sessions, and decide what happens next. Use when Mohmed asks 'what's next', 'plan this', 'break this down', 'how do we tackle this', when a phase is starting, when a phase gate has just passed, or when a session needs a scoped task to work on. Also use to decide session boundaries and ordering, and to write the handoff into docs/status.md. Do NOT use for writing the requirements themselves; that is spec-writing. This skill decides ORDER and SESSION BOUNDARIES, not what the software should do."
---

# Phase planning

Turns a phase into a sequence of sessions, each of which is one builder pass with a gate.

## Rules of order

**Riskiest assumption first, not easiest task first.** The purpose of ordering is to fail cheaply. A task that can only fail after four other tasks are built is scheduled first if it is the one likely to fail.

**One vertical slice per session.** A slice touches every layer and produces something demoable. Never a horizontal session that builds "all the routes" or "all the components", because nothing is verifiable until the other half exists.

**Do not start a phase before the previous gate passes.** Not "mostly passes". Ran the command, read the output, pasted it into `status.md`.

**A session that cannot state its gate is not ready to start.** Write the gate first. If you cannot, the task is underspecified and belongs back with `spec-writing`.

## Session shape

Every builder session gets exactly this handed to it:

```
Task: <one vertical slice, one outcome>
Story: <S-id, or the phase deliverable>
Constraint: <the thing that makes this non-obvious>
Avoid: <two or three anti-patterns this task attracts, from slop-catalog>
Gate: <the command that proves it done>
Out of scope: <what the session must not touch>
```

The "out of scope" line prevents the most expensive failure, which is a session refactoring something adjacent that nobody asked it to touch.

## Sequencing across sessions

One builder session, then one separate reviewer session. Never one session doing both. The reviewer needs a context that did not write the code, because the context that wrote it cannot see what it assumed.

Between them, Mohmed's merge. That is the only human gate and it is the last one.

## Sizing

A session should complete in one context window with room left for the review. Signals it is too big:

- More than one story
- More than roughly six files touched
- The gate needs more than three commands
- The task description contains "and also"

Split on the outcome boundary, not the file boundary.

## When a phase is blocked

Do not work around a blocker by starting the next phase. Record it in `status.md` with what would unblock it and who owns that, then pick up a task inside the same phase that the blocker does not touch. If there is none, say so and stop, rather than manufacturing progress.

## When work reveals the plan was wrong

This is normal, not a failure. Update `phases.md`, note the change in its revision log with the reason, and continue. Do not silently deviate from a written phase, because the next session will read the doc and not the deviation.

## Handoff

Every session ends with `status.md` updated: what moved to Done, the rewritten Next list with a genuine top item, any new blocker, and a session log line describing state rather than intent.

"Started the webhook handler" is useless. "Webhook route verifies signature and dedupes, fulfillment not written, test file exists and fails" is a handoff.
