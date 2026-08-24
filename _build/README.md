# _build

**This folder builds the template. It is deleted before the template is tagged, and it never exists in a project cloned from the template.**

Everything else in this repo is permanent. This folder is not.

## What is in here

| File | Purpose |
|---|---|
| `CLEANUP.md` | The exact delete list and the order to run it |
| `phase-0.md` through `phase-7.md` | One builder session each. Construction detail for the template's own code |

## How it works

`docs/phases.md` states the gates. These files state the construction: which files to create, what properties they must have, and which anti-patterns each task attracts.

One session per file. Read `CLAUDE.md`, the rules, the skill named in the file's Skills line, and then the file. Nothing else.

After each session:

1. Run the gate. Paste the actual output into the PR.
2. Open the PR. Mohmed merges.
3. Update `docs/status.md`.
4. A separate reviewer session reviews before merge. Never the same session that built it.

## Do not

- Do not treat these as the project's phases. They build the template. A cloned project runs `project-init` and then follows `docs/phases.md` for its own work.
- Do not skip a gate to move faster. A phase with a skipped test is not complete.
- Do not delete this folder before the phase 7 gate passes.
