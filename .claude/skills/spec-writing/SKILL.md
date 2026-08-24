---
name: spec-writing
description: "Write and maintain requirements and user stories in docs/product.md. Use when Mohmed asks for a spec, requirements, user stories, acceptance criteria, or says 'what exactly should this do', 'write this up', 'scope this feature', or when a task is about to start without testable criteria. Also use when deciding what is above or below the MVP line, or when a story has grown too large and needs splitting. Do NOT use for breaking a written story into tasks and sessions; that is phase-planning. This skill decides WHAT the software must do, phase-planning decides in what order it gets built."
---

# Spec writing

A story without observable acceptance criteria is a wish. This skill exists to stop wishes reaching a builder session.

## Story shape

```
### S-<id> <short title>

As a <role>, I want <capability>, so that <outcome>.

Acceptance criteria:
- Given <state>, when <action>, then <observable result>.

Edge cases:
- <what happens when it goes wrong>

Out of scope for this story:
- <the thing someone will assume is included>
```

The "out of scope" field is not optional. It prevents the most common failure, which is a builder session quietly expanding the story.

## What makes a criterion valid

- **Observable.** Someone outside the code can see it happen. "The user feels confident" fails. "The submit button is disabled until both fields validate" passes.
- **Binary.** It either happened or it did not. No "should generally".
- **Attributable to one actor.** If a criterion needs two people to do two things, it is two criteria.

Every story touching money, authentication, or deletion states its failure path explicitly. Not the happy path plus a shrug.

## Sizing

A story that cannot be demonstrated in under two minutes is too big. Split by:

1. **Outcome, not layer.** Never "build the API" then "build the UI". Both halves are undemoable alone.
2. **Happy path first, then each failure branch.** The branches are frequently larger than the path.
3. **One user, one goal, one session.**

If splitting produces a story with no user-visible outcome, the split was on a layer boundary. Try again.

## The MVP line

Everything above the line ships before launch. Everything below moves to `docs/product.md` with a promotion trigger.

Moving an item above the line after launch planning has started requires one of: a change to the charter's success metric, or an accepted schedule slip. Write which one in the revision log. An item that silently crosses the line is scope creep with a paper trail conveniently missing.

## Non-functional requirements are stories too

Performance, accessibility, and error budgets live in the same doc with the same gate discipline. A budget with no measurement method is decoration. Name the tool.

## Before writing

Read `docs/product.md`. A story that does not serve the charter's one line or its success metric does not belong above the MVP line, no matter how reasonable it sounds in isolation. Say that plainly when it happens.

## Open questions

A story that cannot be written because something is undecided goes to the Open Questions table with a named owner, not into the story list with an assumption baked in. Assumptions written as requirements are how the wrong thing gets built correctly.
