---
name: security-review
description: "Run a security pass over code or a design. Use before any deploy, before merging anything touching auth, payments, or data access, when Mohmed asks 'is this safe', 'security review', 'did I miss anything', 'can someone abuse this', or at the phase 5 gate. Load references/checklist.md for the full pass. Covers authorization gaps, secrets, input handling, dependency risk, and headers, ordered by cost of being wrong. Do NOT use for building an auth flow; that is auth-flows. This skill verifies what was built rather than deciding what to build."
---

# Security review

The job is to find what is wrong, not to confirm it is fine. A pass that returns "looks good" without genuinely looking for failure launders risk into confidence.

Load `references/checklist.md` for the full pass.

## Order findings by cost of being wrong

Report in this order and stop at the first three if present. A design with an authorization gap does not need a naming discussion.

1. **Silent money or data loss.** Payment without access granted, write that fails without surfacing, entitlement that never expires.
2. **Authorization gaps.** Any route where ownership is not verified inside the query.
3. **Secret exposure.** A key in a client bundle, a token in a log, a secret in the repo history.
4. **Input handling.** An unvalidated boundary, an injection path, an unbounded upload.
5. **Failure modes.** Unhandled third-party outage, unbounded retry, swallowed error.
6. **Dependency risk.** Known vulnerabilities, unmaintained packages.
7. **Headers and transport.** CSP, HSTS, cookie flags.

## The test that catches the most

Authenticate as user A. Request user B's resource by id. On every parameterized route.

This finds more real bugs than every other check combined, because it is the one failure that looks completely normal in the happy path. If it passes trivially, verify the route exists before believing the result.

## Threat model discipline

Defend against what actually happens to a small application, in this order: automated scanners, credential stuffing, a curious paying user editing an id, a freeloader calling an action directly, payment fraud, spam.

Not in scope at this size: targeted attackers, insider threat, nation-state. Say so explicitly, so effort lands where the risk is rather than where the drama is.

## Findings shape

Each finding names three things:

1. **The mechanism.** What specifically goes wrong, in what sequence. Not "this could be a problem."
2. **The trigger.** What has to happen for it to bite. If the answer is "nothing, it is already broken," say that first.
3. **The fix, sized.** One-line change, refactor, or redesign.

A finding without a mechanism is a vibe. Without a trigger it cannot be prioritized. Without a size it cannot be scheduled.

## Reviewing your own work

The failure mode is defending rather than testing. Re-run the checks against your own code with the same hostility. Withdrawing your own suggestion costs one sentence; leaving a hole in costs an incident.

## Gate

```
pnpm audit --audit-level=moderate
semgrep --config=auto
pnpm test -- authz
```

Plus the authorization matrix in `architecture.md` verified row by row against the actual code. A matrix that has drifted from the code is worse than no matrix.
