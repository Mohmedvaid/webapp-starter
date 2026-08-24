# Status

> Read this first in every session. Update it last.
>
> This file exists because context does not survive. It is the handoff between sessions, between machines, and between you and any agent working in this repo. A stale status file is worse than no status file, because it is trusted.
>
> Keep it short. If it grows past two screens, the detail belongs in the doc that owns it.

**Last updated:** 2026-08-24, template audit and scaffold session

---

## Right now

**Current phase:** Phase 0, Foundation
**Working on:** nothing. Docs, skills, and repo scaffolding are in place. There is no `package.json`, no `src/`, no application code.
**Blocked by:** owner calls listed under Open decisions. Phase 0 can start; the gate-composition call should be made before that session writes `pnpm gate`.

## Done

Most recent first. One line each. Prune anything older than the current phase into the phase table in `phases.md`.

| Date | What | Evidence | PR |
|---|---|---|
| 2026-08-24 | Scaffold: `.gitignore`, `.nvmrc` (22.23.2), `.editorconfig`, `AGENTS.md` symlink to `CLAUDE.md` | `709692c` | main |
| 2026-08-24 | Docs and skills restored to their CLAUDE.md paths; dead numbered doc names, ADR id collision, and 404/403 mismatch fixed | `886aafd` | main |

## Next

Ordered. Top item is what the next session picks up without asking.

1. Run `_build/phase-0.md`
2. Resolve the `pnpm gate` composition mismatch (Open decisions) before wiring the command
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
| What `pnpm gate` contains | Short list in CLAUDE.md/phases.md vs full list in ADR 0002 (adds integration, Lighthouse, `pnpm audit`, Semgrep) vs growing the set across phases | ADR 0002 is the accepted decision; Lighthouse and Semgrep are phase 5/6 deliverables, so the list may be the end state | Phase 0 |
| Four async states vs five-state matrix | `typescript.md` and `frontend-patterns` omit Partial; `design-system.md` owns states and includes it | Follow `design-system.md` | Phase 1 |
| TypeScript "no enums" vs schema enums | `typescript.md` bans TS enums; db skill and phase 3 require Postgres enums for multi-state | Different layers, same word; say so explicitly or an agent will refuse DB enums | Phase 3 |

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
| 2026-08-24 | Audited docs/skills, fixed dead numbered paths and ADR 0002 collision, added `.gitignore` `.nvmrc` `.editorconfig` `AGENTS.md` → `CLAUDE.md`. No `package.json` or `src/`. | Phase 0 not started. Next session runs `_build/phase-0.md`. |

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
