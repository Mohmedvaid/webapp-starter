# webapp-starter

Runnable Next.js + TypeScript template for web apps. Clone it, run `project-init`, build.

Auth, payments, database, design tokens, env validation, health checks, logging, and CI are wired. A generic Projects feature ships as a working vertical slice and is deleted in one commit once you have your own.

## Run it

```bash
pnpm install
cp .env.example .env        # fill it, the app refuses to boot otherwise
pnpm db:migrate && pnpm db:seed
pnpm dev
```

Then in Claude Code: `use project-init` and answer the questions. It fills the docs, records your database and auth choice, and removes the payment mode you are not using.

## Stack

Next.js App Router, TypeScript strict, pnpm, Node 22. Tailwind and shadcn/ui through design tokens. Drizzle over Postgres. Stripe. Resend. Vitest and Playwright. Vercel.

Swappable: the database, the auth provider, and the UI library are each isolated behind one module. Everything else is load-bearing.

## Where things are

| Path | What |
|---|---|
| `CLAUDE.md` | Always-loaded context. Read it before anything else |
| `.claude/rules/` | Standards that apply to every task |
| `.claude/skills/` | Workflows, loaded when the task matches |
| `docs/` | Charter through status. `status.md` says where the project actually is |
| `src/core/` | Pure domain logic, no framework |
| `src/server/` | Data access and authorization |
| `src/app/` | Routes, no logic |

Imports go one way: `app` to `features` to `server` to `core`. Enforced by lint.

## Working in it

Phases run 0 through 7 in `docs/phases.md`. Each has a gate you run rather than assert. Do not start a phase before the previous gate passes.

`docs/status.md` is the handoff file. Read it first, update it last, every session.

## License

Private. Not for distribution.
