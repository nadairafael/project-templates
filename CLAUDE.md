# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A documentation framework for bootstrapping new projects. No source code — only checklists, decision guides, and templates. The workflow is: copy → decide → fill in → develop.

## How to bootstrap a new project from this repo

Clone this repo as the new project's docs folder:

```bash
git clone <url> docs
```

Or copy the files manually. Suggested destination structure:

```
docs/
├── decisions/              ← copy folder as-is
├── workflow/               ← copy folder as-is
├── architecture/
│   ├── adr/
│   │   └── 000-template.md
│   └── overview.md         ← copy from ARCHITECTURE.md
```

Then follow `workflow/bootstrap.md` phase by phase.

## File map

```
project-templates/
├── decisions/              # Step 1 before writing any code — pick your stack
│   ├── orm.md              # Drizzle vs Prisma
│   ├── runtime.md          # Cloudflare vs Vercel vs Railway
│   ├── database.md         # D1 vs Postgres vs Turso
│   ├── auth.md             # better-auth vs Clerk vs Auth.js vs Supabase
│   ├── ui.md               # Tailwind + shadcn vs custom vs CSS Modules + referências visuais
│   ├── testing.md          # Vitest vs Jest; Playwright vs Cypress
│   ├── api-style.md        # REST vs tRPC vs GraphQL vs Server Actions
│   ├── background-jobs.md  # Trigger.dev vs Inngest vs BullMQ vs CF Queues
│   ├── file-storage.md     # R2 vs S3 vs Uploadthing vs Supabase Storage
│   ├── email.md            # Resend vs Postmark vs SES vs SendGrid
│   ├── error-monitoring.md # Sentry vs Axiom vs Highlight
│   ├── realtime.md         # SSE vs Supabase RT vs PartyKit vs WebSocket
│   ├── search.md           # Postgres FTS vs Meilisearch vs Typesense vs Algolia
│   ├── ai.md               # Vercel AI SDK vs SDK direto vs LangChain
│   └── monorepo.md         # Sem monorepo vs Turborepo vs Nx
│
├── workflow/
│   ├── bootstrap.md        # Entry point — 5 phases, do not skip
│   ├── project-context.md  # Product context for the LLM — personas, routes, flows, visual level
│   ├── conventions.md      # Code conventions skeleton (fill in after decisions)
│   ├── new-feature.md      # Per-feature implementation checklist
│   ├── schema-drizzle.md   # Schema change workflow for Drizzle projects
│   └── schema-prisma.md    # Schema change workflow for Prisma projects
│
├── architecture/
│   └── adr/000-template.md # ADR template for architectural decisions
│
├── ARCHITECTURE.md          # Blank overview template — fill in as project grows
├── README.md
└── CLAUDE.md
```

## How files connect

```
bootstrap.md
  ├── Phase 2 → decisions/         (pick ORM, runtime, DB, auth, UI + visual refs)
  ├── Phase 2 → adr/000-template   (document each choice as an ADR)
  ├── Phase 3 → Context7           (fetch up-to-date docs for chosen libs)
  ├── Phase 3 → conventions.md     (fill in with chosen stack's conventions)
  ├── Phase 3 → scaffold           (initialize project based on Context7 docs)
  ├── Phase 4 → new-feature.md     (one run per feature)
  │               └── schema changed? → schema-drizzle.md or schema-prisma.md
  └── Phase 5 → ARCHITECTURE.md   (document what was actually built)
```

## Decision guides — format and intent

Each `decisions/*.md` file follows the same structure so they can be scanned quickly:

1. **Options** — each with "choose when", trade-offs, and "what follows" (conventions, files, commands)
2. **Summary table** — side-by-side comparison
3. **Quick rule** — one-liner for the most common case

The goal is decision-first, not prescription. A project using Drizzle follows `schema-drizzle.md`; one using Prisma follows `schema-prisma.md`. Both are valid.

## When editing files in this repo

- `decisions/*.md` — update when a new library becomes the better default or trade-offs change
- `workflow/conventions.md` — keep agnostic; add ORM/UI sections in pairs (one per option)
- `workflow/bootstrap.md` — minimal edits; it is the stable entry point
- `ARCHITECTURE.md` — keep as a blank template; do not fill it with project-specific content
