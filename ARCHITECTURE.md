# Architecture Overview

_Template a ser preenchido durante o bootstrap do projeto. Começa com as decisões tecnológicas (Fase 2 do `workflow/bootstrap.md`) e vai sendo completado conforme o projeto evolui. Seções marcadas com `_definir_` devem ser preenchidas._

---

## Stack

### Core

| Camada | Tecnologia | Guia de decisão |
|--------|-----------|-----------------|
| Framework | _definir_ (ex: React Router v7, Next.js 15, TanStack Start) | — |
| Runtime / Deploy | _definir_ (ex: Cloudflare Workers + Pages, Vercel, Railway) | `decisions/runtime.md` |
| Banco de dados | _definir_ (ex: Cloudflare D1, Neon, Supabase, Turso) | `decisions/database.md` |
| ORM | _definir_ (ex: Drizzle, Prisma) | `decisions/orm.md` |
| Auth | _definir_ (ex: better-auth, Clerk, Auth.js) | `decisions/auth.md` |
| API style | _definir_ (ex: REST, tRPC, Server Actions) | `decisions/api-style.md` |
| UI | _definir_ (ex: Tailwind v4 + shadcn/ui, Tailwind v4 + custom) | `decisions/ui.md` |
| Forms | _definir_ (ex: @conform-to/react + Zod, React Hook Form + Zod) | — |
| Testing | _definir_ (ex: Vitest + Playwright, Jest + Cypress) | `decisions/testing.md` |
| Linting | _definir_ (ex: Biome, ESLint + Prettier) | — |
| Package manager | _definir_ (ex: bun, pnpm, npm) | — |

### Opcional — adicionar conforme o projeto precisar

| Camada | Tecnologia | Guia de decisão |
|--------|-----------|-----------------|
| i18n | _definir_ (ex: next-intl, i18next + react-i18next) | — |
| Email | _definir_ (ex: Resend, Postmark, AWS SES) | `decisions/email.md` |
| Mensageria | _definir_ (ex: Evolution API, Twilio) | — |
| Pagamentos | _definir_ (ex: Stripe, LemonSqueezy, Paddle) | — |
| Background jobs | _definir_ (ex: Trigger.dev, Inngest, BullMQ) | `decisions/background-jobs.md` |
| File storage | _definir_ (ex: Cloudflare R2, S3, Uploadthing) | `decisions/file-storage.md` |
| Real-time | _definir_ (ex: SSE, Supabase Realtime, PartyKit) | `decisions/realtime.md` |
| Search | _definir_ (ex: Postgres FTS, Meilisearch, Algolia) | `decisions/search.md` |
| AI | _definir_ (ex: Vercel AI SDK, Anthropic SDK) | `decisions/ai.md` |
| Error monitoring | _definir_ (ex: Sentry, Axiom, Highlight) | `decisions/error-monitoring.md` |
| Monorepo | _definir_ (ex: Turborepo, Nx) ou N/A | `decisions/monorepo.md` |

---

## Arquitetura em camadas

_Adicionar diagrama após definir runtime e banco. Exemplo de estrutura:_

```
Browser
  └─ [Framework] (hydration + cache client)
        │
[Runtime / CDN]
  └─ [Servidor / Worker]
        │
        ├─ Middleware (auth, i18n, contexto de tenant)
        ├─ Handlers / Loaders / Actions
        ├─ Repositories / Services
        └─ [Banco de dados]
```

---

## Estrutura de diretórios

_Preencher após scaffold inicial. Documentar apenas o que não é óbvio pela estrutura._

```
/
├── _definir_
```

---

## Rotas

_Preencher conforme rotas são criadas. Manter atualizado com status de implementação._

### Públicas

| URL | Arquivo / Handler |
|-----|------------------|
| `/` | _definir_ |

### Autenticadas

| URL | Arquivo / Handler | Status |
|-----|------------------|--------|
| _definir_ | _definir_ | — |

### APIs

| URL | Descrição |
|-----|-----------|
| `/api/auth/*` | Auth handler |

---

## Modelagem de dados

_Preencher após decisão de banco e ORM. Documentar entidades principais, relações e decisões não-óbvias._

### Decisões de modelagem a documentar

- Como o multi-tenancy é implementado (se aplicável)
- Como valores monetários são armazenados
- Estratégia de soft delete vs hard delete
- Campos de snapshot histórico (ex: copiar nome do cliente na invoice)
- Relações com `set null` vs `cascade` e o motivo

---

## Padrões e convenções

_Preencher conforme padrões emergem durante o MVP (Fase 4 do bootstrap.md). Não inventar no dia 1 — documentar o que realmente existe no código._

### IDs

_definir_ — **recomendado: UUID v7** (`uuidv7()`) — ordenável cronologicamente, sem colisão, bom para paginação por cursor. Alternativas: CUID2 (mais curto, URL-safe), UUID v4 (aleatório), auto-increment (simples, mas expõe volume).

### Dinheiro

_definir_ — **recomendado: dois campos por valor** — `amount` (integer em centavos) + `amountCurrency` (string ISO 4217, ex: `"BRL"`). Evita arredondamento de float e suporta multi-moeda nativamente. Alternativa: `Decimal(15,2)` se o banco suportar (não funciona em SQLite/D1).

### Timestamps

_definir_ — **recomendado: milliseconds como integer** (`Date.now()`) — compatível com edge (sem timezone issues), ordenável, fácil de serializar. Alternativa: ISO string se legibilidade importa; `DateTime` do Prisma se usando Postgres.

### Multi-tenancy

_definir_ ou N/A — **padrão recomendado:** `organizationId` em toda tabela de negócio com `onDelete: cascade`. Contexto da org ativa injetado via middleware em rotas privadas — nunca confiar no corpo do request.

### Acesso ao banco

_definir_ — **recomendado: repository pattern** (`app/repositories/*.repository.ts`) — isola queries do Drizzle/Prisma dos handlers, facilita testes e troca de banco. Alternativa: queries diretas nos loaders/actions para projetos pequenos (menos indireção).

### Navegação interna

_definir_ — **recomendado: `href()` helper** do React Router / framework para type-safety de rotas em tempo de compilação. Alternativa: objeto de constantes `ROUTES = { clients: "/clients" }` centralizado.

### Formulários

_definir_ — ver seção Formulários em `workflow/conventions.md`. **Recomendado para SSR:** `@conform-to/react` + Zod (progressive enhancement). **Recomendado para client-side:** React Hook Form + Zod.

### i18n

_definir_ ou N/A — **recomendado para Next.js:** `next-intl` (type-safe, Server Components). **Recomendado para React Router / agnóstico:** `i18next` + `react-i18next`. Strings de UI nunca hardcoded — mesmo projetos em idioma único devem centralizar strings para facilitar mudanças.

### Testing

_definir_ — documentar onde os testes vivem e o que cada camada cobre:

| Camada | Localização | O que cobre |
|--------|-------------|-------------|
| Unit | `tests/unit/` ou `*.test.ts` ao lado do arquivo | Funções puras, schemas Zod, utils |
| Integration | `tests/integration/` | Repositories, handlers com DB |
| E2E | `e2e/` | Fluxos críticos no browser |

_Preencher após decisão de ferramenta (ver `decisions/testing.md`)._

---

## Gotchas e decisões táticas

_Adicionar durante a Fase 4 (Implementar MVP). Qualquer comportamento inesperado do framework/lib ou decisão pequena que não justifica ADR mas precisa ser lembrada._

| Contexto | Gotcha / Decisão |
|----------|-----------------|
| _ex: D1 + Drizzle_ | _ex: `getDb(context)` deve ser chamado a cada request — sem singleton em Workers_ |
