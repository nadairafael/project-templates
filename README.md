# Project Templates

Framework de documentacao para bootstrap de projetos novos. A ideia central: tomar decisoes tecnologicas de forma informada (com trade-offs explícitos) antes de escrever codigo, e manter essa documentacao viva durante o desenvolvimento.

## Como usar

Clone este repo como a pasta de docs do novo projeto:

```bash
git clone <url> docs
```

Ou copie os arquivos que precisar para dentro do seu projeto. Estrutura sugerida no destino:

```
docs/
├── decisions/          ← copiar pasta inteira
├── workflow/           ← copiar pasta inteira
├── architecture/
│   ├── adr/
│   │   └── 000-template.md
│   └── overview.md     ← copiar de ARCHITECTURE.md
```

Depois seguir `workflow/bootstrap.md` fase por fase:

```
Fase 1 → definir o que o sistema faz e seus non-goals
Fase 2 → consultar decisions/ e criar ADRs para cada escolha fundacional
Fase 3 → buscar docs via Context7, preencher conventions.md e fazer scaffold
Fase 4 → implementar o MVP com new-feature.md por feature
Fase 5 → consolidar a documentacao com o que foi realmente construido
```

## Conteudo

### `decisions/` — guias de decisao tecnologica

Consultar na Fase 2 do bootstrap. Cada arquivo descreve opcoes, trade-offs e o que segue de cada escolha.

| Arquivo | Cobre |
|---------|-------|
| `decisions/orm.md` | Drizzle vs Prisma |
| `decisions/runtime.md` | Cloudflare Workers vs Vercel vs Railway |
| `decisions/database.md` | D1 vs PostgreSQL (Neon / Supabase / Railway) vs Turso |
| `decisions/auth.md` | better-auth vs Clerk vs Auth.js vs Supabase Auth |
| `decisions/ui.md` | Tailwind + shadcn vs Tailwind + custom vs CSS Modules + referências visuais |
| `decisions/testing.md` | Vitest vs Jest; Playwright vs Cypress |
| `decisions/api-style.md` | REST vs tRPC vs GraphQL vs Server Actions |
| `decisions/background-jobs.md` | Trigger.dev vs Inngest vs BullMQ vs CF Queues |
| `decisions/file-storage.md` | R2 vs S3 vs Uploadthing vs Supabase Storage |
| `decisions/email.md` | Resend vs Postmark vs AWS SES vs SendGrid |
| `decisions/error-monitoring.md` | Sentry vs Axiom vs Highlight vs LogSnag |
| `decisions/realtime.md` | SSE vs Supabase Realtime vs PartyKit vs WebSocket |
| `decisions/search.md` | Postgres FTS vs Meilisearch vs Typesense vs Algolia |
| `decisions/ai.md` | Vercel AI SDK vs SDK direto vs LangChain |
| `decisions/monorepo.md` | Sem monorepo vs Turborepo vs Nx |

### `workflow/` — checklists de processo

| Arquivo | Descricao | Adaptar? |
|---------|-----------|----------|
| `bootstrap.md` | Checklist de 5 fases para iniciar um projeto do zero | Nao |
| `project-context.md` | Contexto do produto para o agente — personas, rotas, fluxos, nivel visual | Sim |
| `conventions.md` | Convencoes de codigo — esqueleto para preencher apos as decisoes | Sim |
| `new-feature.md` | Checklist para implementar uma feature (scope → build → verify → doc) | Sim (camadas) |
| `schema-drizzle.md` | Mudanca de schema com Drizzle + D1/Turso/Postgres | Sim (dependentes) |
| `schema-prisma.md` | Mudanca de schema com Prisma + Postgres | Sim (dependentes) |

### `architecture/`

| Arquivo | Descricao |
|---------|-----------|
| `adr/000-template.md` | Template de ADR (Architecture Decision Record) |

### `ARCHITECTURE.md`

Template de visao geral da arquitetura. Começa com campos `_definir_` e vai sendo preenchido conforme o projeto evolui.
