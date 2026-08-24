---
name: docs-maintenance
description: "Keep the docs true. Use after any shipped feature, at the end of every session, when a decision gets made, when a doc contradicts the code, or when about to create a new document. Triggers include 'update the docs', 'document this', 'where does this go', 'should I write this down', 'the docs are out of date', or finishing any piece of work. Enforces update-do-not-duplicate, the ownership map, and the discipline that docs/status.md describes state rather than intent. Creating a new doc when an existing one owns the topic is the specific failure this skill exists to prevent."
---

# Docs maintenance

Two rules carry almost everything: **update, never duplicate**, and **describe state, not intent**.

## Ownership map

Before writing anything, find the doc that owns the topic. There is always one.

| Topic | Doc |
|---|---|
| What the product is, who for, why, stories, criteria, MVP line, backlog | `product` |
| Stack, boundaries, data model, threat model, authorization matrix, decisions | `architecture` plus `adr/` |
| Tokens, components, states, accessibility | `design-system` |
| Order of work, gates | `phases` |
| Env, runbook, rollback, monitoring, incidents | `operations` |
| Right now | `status` |

If a topic seems to fit nowhere, it almost certainly belongs in one of these as a new section. A genuinely new document is rare and is proposed rather than created.

## Do not create a new doc

This has been corrected more than once and it is worth restating plainly. When tempted to create a new file, the answer is a new section in the doc that owns the topic.

A second doc on a topic guarantees the two will disagree, and the reader has no way to tell which is current.

The concrete failure: a QA document kept its own list of issues, marked two of them fixed, and both were live in production at the time. The same pattern produced three overlapping work lists in three vocabularies. A QA log, a punch list, or a TODO file living outside `status.md` **is** the duplicate. There is one work list.

## When a doc contradicts the code

The doc is wrong until proven otherwise, because the code runs and the doc does not. Fix the doc, note it in the revision log with the reason, and consider whether the drift means a gate is missing.

If the code is the one that is wrong, that is a bug with a doc as its bug report. Fix the code.

## ADRs

Write one when a decision is made that a future session would otherwise silently reverse: a library choice, a boundary, a tradeoff accepted deliberately.

Use the `0001` format. Short. Context, decision, consequences including the bad ones, alternatives and why they lost, and when to revisit.

Do not write an ADR for a decision with no alternative. That is not a decision, it is a fact.

## Status discipline

`status.md` is read first in every session and updated last. It is the handoff between sessions, machines, and people.

At the end of every session:

1. Move finished work to Done, with the PR.
2. Rewrite Next so the top item is genuinely the next action, not a category.
3. Add or clear blockers.
4. Add a session log line describing state.
5. Stamp the date.

Reconcile against code and production, not against the last update. A status doc updated from a previous status doc drifts monotonically and never corrects.

A claim that something is fixed carries evidence: a commit sha, a test name, or a production check that was actually run. Without one it is an intention wearing the past tense.

Describe state, not intent. "Started the webhook handler" is useless. "Webhook route verifies signature and dedupes, fulfillment not written, test file exists and fails" is a handoff.

## Revision logs

Every doc has one. One line per meaningful change: date, what changed, why. The why is the part that pays off in six months, because the diff already shows the what.

## Pruning

Docs grow. At each phase gate, delete what is no longer true, move completed detail into the phase table, and collapse anything that has become historical. A doc nobody trusts because it is stale is worse than a missing doc, because it gets read.
