# Product

> Template. `project-init` fills the FILL markers by interviewing the owner. Do not build until the charter section is filled.
> This doc owns what the product is, what it must do, and what is deliberately not being done. Implementation detail belongs in `architecture.md`.

---

# Part 1: Charter

## One line

<!-- FILL: one sentence a stranger understands. What it does, for whom. -->

## The problem

<!-- FILL: what breaks today without this. Be specific about the moment of pain, not the market category. -->

## Who it is for

<!-- FILL: the primary user. One segment, not three. Name the second segment only if the MVP serves it. -->

**Primary user:**
**They currently solve this by:**
**Why that is inadequate:**

## Why this can win

<!-- FILL: the durable advantage. Not "better UX". Something a competitor cannot copy in a week. -->

## Success metric

One number. If it moves, the project is working. If it does not, nothing else matters.

**Metric:**
**Current value:**
**Target and by when:**

## Monetization

| Field | Value |
|---|---|
| Model | <!-- FILL: one-time, subscription, usage, free --> |
| Price | |
| What the price anchors against | |
| What gates the paywall | |
| Refund policy | |

## Non-goals

Things this explicitly does not do. This list prevents scope creep more effectively than any process.

- <!-- FILL -->
- <!-- FILL -->
- <!-- FILL -->

## Constraints

| Constraint | Value |
|---|---|
| Owner time per week | |
| Budget ceiling | |
| Hard deadline, if any | |
| Compliance or regulatory scope | |

## Exit condition

What "done" looks like, and what happens after.

<!-- FILL: sold, sunset, maintained indefinitely, folded into something else. -->

---

# Part 2: Requirements

## Non-functional budgets

These are gates, not aspirations. A build that violates one does not ship.

A budget that is only measured after release is not a gate, it is a report. Each row names the check that fails the build, and Lighthouse runs against a production build rather than the dev server, since dev numbers are meaningless.

| Budget | Target | Measured by |
|---|---|---|
| Page load, p75 | LCP under 2.5s | Lighthouse in `pnpm gate`, against a production build |
| Interaction | INP under 200ms | Lighthouse in `pnpm gate` |
| API response, p95 | under 500ms | Server logs |
| Accessibility | WCAG 2.1 AA on every shipped page | axe in `pnpm gate` |
| Error rate | under 0.5% of requests | Error monitor |
| Uptime | <!-- FILL --> | Uptime check |

## Story format

Every story uses this shape. A story without testable acceptance criteria is not a story, it is a wish.

```
### S-<id> <short title>

As a <role>, I want <capability>, so that <outcome>.

Acceptance criteria:
- Given <state>, when <action>, then <observable result>.
- Given <state>, when <action>, then <observable result>.

Edge cases:
- <what happens when it goes wrong>

Out of scope for this story:
- <the thing someone will assume is included>
```

Rules:
- Criteria are observable. "The user feels confident" is not a criterion. "The button is disabled until both fields validate" is.
- Every story that touches money, auth, or data deletion names its failure path explicitly.
- A story that cannot be demoed in under two minutes is too big. Split it.

---

## MVP

Everything above the line ships before launch. Everything below does not.

### Epic: Account and access

<!-- FILL: stories -->

### Epic: <core value delivery>

<!-- FILL: stories. This is the epic that delivers the charter's one line. -->

### Epic: Payment and entitlement

<!-- FILL: stories -->

### Epic: Operations

<!-- FILL: stories the owner needs, not the user: seeing signups, seeing revenue, handling a refund -->

---

## MVP LINE

Nothing below this point is built before launch. Moving an item above the line requires updating the success metric in `product.md` or accepting a slip, and writing which in the revision log.

---

## Explicit out of scope

Named so nobody re-proposes them. Each with the reason.

| Not building | Why not |
|---|---|
| <!-- FILL --> | |

## Open questions

Things that block a story from being written. Each with an owner and a date.

| Question | Blocks | Owner | Raised |
|---|---|---|---|
| | | | |

---

# Part 3: Backlog

Everything below the MVP line. Ordered. Every item carries the reason it is waiting and the trigger that promotes it. An item with no promotion trigger is not a backlog item, it is a daydream. Delete it or give it a trigger.

Promotion means moving the story up into Part 2 above the MVP line. Delete it from here. Never duplicate.

## Format

```
### B-<id> <short title>

What: <one line>
Why not now: <the actual reason, not "no time">
Promotes when: <a condition you can observe, not a feeling>
Rough size: S / M / L
Depends on: <items or phases>
```

---

## Queue

<!-- FILL: seeded by project-init from the charter conversation, then maintained by phase-planning -->

### B-001 <title>

What:
Why not now:
Promotes when:
Rough size:
Depends on:

---

## Parked

Considered and set aside. Kept so the same idea does not get re-litigated every month.

| Item | Considered on | Set aside because | Would revisit if |
|---|---|---|---|
| | | | |

## Killed

Decided against permanently. Kept as a record so the decision is not silently reversed.

| Item | Killed on | Reason |
|---|---|---|
| | | |

---

## Revision log

| Date | Change | Why |
|---|---|---|
| | Created from template | |
