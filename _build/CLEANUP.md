# Cleanup

Two deletions with different lifetimes. Do not confuse them.

---

## Deletion 1: once, when the template is finished

Runs in the template repo, one time, in the final commit before tagging v1. After this, the template is done and every clone starts clean.

**Trigger:** the phase 7 gate in `docs/phases.md` has passed.

**Delete:**

```
_build/                    the whole folder, including this file
```

That is the entire list. Everything else in the repo ships to every clone.

**Also do, in the same commit:**

- Reset `docs/status.md` to its empty template state. The template's own build history is not a clone's starting status.
- Confirm `docs/product.md` still has its FILL markers intact. If the template's own build filled them in, restore them.
- Tag `v1`.

**Commit:**

```
chore: remove build scaffolding, tag template v1
```

---

## Deletion 2: per clone, when the project has a real feature

Runs in each project cloned from the template, not in the template itself.

**Trigger:** the project's first real feature slice is merged and passing.

**Delete:**

```
src/features/projects/           the reference vertical slice
tests/e2e/projects.spec.ts
tests/unit/projects/
src/db/seed/projects.ts          the seed rows for it
```

Then remove the route from the navigation, drop the `projects` table in its own migration, and remove any lingering imports.

**Why it ships at all:** it is the working pattern Claude Code copies. An abstract description of the layering produces a worse first feature than a concrete example does. Delete it once a real one exists, not before.

**Also delete at clone time, during `project-init`:**

- The unused payment mode. Keeping one-time means removing the subscription handlers, price handling, and billing portal route. Keeping subscription means removing the single-purchase entitlement path.
- Any env variable in `src/lib/env.ts` and `.env.example` belonging to a service this project does not use.

---

## Never delete

For clarity, because the question comes up:

| Keep forever | Why |
|---|---|
| `CLAUDE.md` | Always-loaded context |
| `.claude/rules/` | Standards on every task |
| `.claude/skills/` | The workflows. Every clone uses them |
| `docs/` | Rewritten per clone by `project-init`, never removed |
| `docs/adr/0001` | The layering decision. Still true in every clone |
| `docs/adr/0002` | One gate command. Still true in every clone |
| `README.md` | Rewritten per clone, never removed |
