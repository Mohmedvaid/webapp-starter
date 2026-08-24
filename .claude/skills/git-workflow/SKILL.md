---
name: git-workflow
description: "Branching, commits, and pull requests. Use when starting work that needs a branch, writing a commit message, opening or describing a pull request, or deciding how to split work across commits. Triggers include 'branch', 'commit', 'PR', 'pull request', 'push this', 'what should the commit message be', 'how do I split this'. Covers naming, conventional commit format, PR contents, and the builder-then-reviewer sequencing. Mohmed's merge is the only human gate and there is no automated branch protection."
---

# Git workflow

## Branches

```
<type>/<short-kebab-description>
```

Types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`.

One branch per vertical slice, matching one builder session. A branch accumulating three unrelated changes cannot be reviewed or reverted cleanly, which is the entire value of a branch.

Branch from `main`, merge to `main`. No long-lived develop branch, because a solo builder with a release branch has invented a merge conflict generator.

## Commits

Conventional commits. The prefix is not decoration, it drives the changelog.

```
feat(payments): add webhook dedupe table
fix(auth): preserve intended destination through login
chore(deps): bump drizzle to 0.31
```

- Subject under 72 characters, imperative mood, no trailing period.
- Body explains why, when why is not obvious. The diff already says what.
- One logical change per commit. "Fix bug and refactor unrelated module" is two commits.
- Never commit commented-out code or a stray debug statement. Both survive longer than intended.

## Pull requests

Every PR carries:

```
## What
One or two lines.

## Why
Story id or phase deliverable.

## Gate
The commands, with their actual output pasted.

## Notes
Anything the reviewer would otherwise have to discover.
```

The pasted gate output is the part that matters. A claim that tests pass is not evidence that tests pass.

## Sequencing

1. Builder session does the work on a branch, opens the PR with gate output.
2. **Separate** reviewer session reviews from a fresh context.
3. Builder session addresses findings.
4. Mohmed merges.

Never one session doing both build and review. The reviewer needs a context that did not write the code.

Mohmed's merge is the only human gate. There is no automated branch protection and no bot approval, deliberately, because a solo builder gains nothing from a gate that only he can satisfy.

State the tradeoff rather than leaving it implied. The merge gate catches what a human reading a diff catches. It does not catch what runs differently on another machine, what a stale branch is missing, or what nobody remembered to run. `pnpm gate` is what covers that, which is why it must be one command and why CI runs that same command rather than a parallel list that can drift. See `docs/adr/0002-verification-gate.md`.

## Never

- Never force push a branch someone else has reviewed.
- Never merge with a failing or skipped test.
- Never commit a secret. If one lands, rotate it first, then clean history. Removing it from the tip does nothing.
- Never rewrite history on `main`.
- Never let a branch live longer than a session's work. Long branches diverge, and the merge cost lands on the person least able to reason about it.

## Reverting

Revert first, diagnose after. A revert is cheap and reversible; leaving a broken `main` while investigating is neither.
