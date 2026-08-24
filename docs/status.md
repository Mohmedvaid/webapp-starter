# Status

> Read this first in every session. Update it last.
>
> This file exists because context does not survive. It is the handoff between sessions, between machines, and between you and any agent working in this repo. A stale status file is worse than no status file, because it is trusted.
>
> Keep it short. If it grows past two screens, the detail belongs in the doc that owns it.

**Last updated:** <!-- FILL: date, and by whom or which session -->

---

## Right now

**Current phase:** Phase 0, Foundation
**Working on:**
**Blocked by:**

## Done

Most recent first. One line each. Prune anything older than the current phase into the phase table in `phases.md`.

| Date | What | Evidence | PR |
|---|---|---|
| | Repo created from template | | |

## Next

Ordered. Top item is what the next session picks up without asking.

1. <!-- FILL -->
2.
3.

## Blockers

Anything stopping work, with what would unblock it and who owns that.

| Blocker | Unblocks when | Owner | Since |
|---|---|---|---|
| | | | |

## Open decisions

Things waiting on a call from the owner. An agent must not decide these unilaterally; surface them and stop.

| Decision | Options | Leaning | Needed by |
|---|---|---|---|
| | | | |

## Known debt

Deliberate shortcuts, so they stay deliberate rather than becoming folklore.

| Debt | Taken because | Costs us | Repay when |
|---|---|---|---|
| | | | |

## Gate evidence

Latest result for each phase gate. Paste the actual command output, not a summary of it.

| Phase | Gate run | Result |
|---|---|---|
| 0 | | |

## Session log

Append one line per working session. This is the trail that makes a cold start possible.

| Date | Session did | Left it at |
|---|---|---|
| | | |

---

## How to update this file

At the end of every session, before the context is gone:

1. Move anything finished into Done, with the PR and the evidence. A claim that something is fixed carries a commit sha, a test name, or a production check that was actually run. Without one it is an intention in the past tense.
2. Rewrite Next so the top item is genuinely the next action, not a category.
3. Add any new blocker, and remove any that cleared.
4. Add a Session log line saying what state the repo is actually in.
5. Update Last updated.

Reconcile against code and production, not against the previous version of this file. A status doc updated from a status doc drifts in one direction and never corrects. Two criticals were once marked fixed here while both were live.

This is the only work list. A QA log, a punch list, or a TODO file living anywhere else is the duplicate, and it is how three overlapping lists in three vocabularies appear.

Do not describe intentions. Describe state. "Started the webhook handler" is useless. "Webhook route verifies signature and dedupes; fulfillment not written; test file exists and fails" is a handoff.
