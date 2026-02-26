# Decisão: ORM

Escolha o ORM antes de iniciar o schema. Muda convenções, comandos e workflow inteiro.

## Opções

### Drizzle

**Escolha quando:**
- Runtime é edge (Cloudflare Workers, Bun)
- Banco é D1, Turso, ou SQLite
- Equipe prefere sintaxe próxima de SQL
- Bundle size importa (zero overhead de geração em build time)

**Trade-offs:**
- Mais verboso que Prisma em queries complexas
- Tooling menos maduro (sem Studio, sem visualizador de schema nativo)
- Migrations geradas via `drizzle-kit generate` — você revisa o SQL antes de aplicar

**Segue com:**
- Schema em `database/schema.ts`
- Migrations em `drizzle/`
- Helper `getDb(context)` para acessar instância (necessário em edge — sem singleton)
- Workflow de schema: ver `workflow/schema-drizzle.md`
- Helpers de schema: `money(fieldName)` (amount + amountCurrency), `timestamps()` (createdAt + updatedAt em ms), `id()` (UUID v7)

---

### Prisma

**Escolha quando:**
- Runtime é Node.js tradicional (Railway, Render, Vercel serverless)
- Banco é PostgreSQL ou MySQL
- Equipe prioriza DX e tooling (Prisma Studio, autocomplete rico, type safety mais explícito)
- Projeto tem muitas relações e queries complexas

**Trade-offs:**
- Client gerado adiciona overhead (ruim para edge / cold start)
- Prisma Accelerate resolve edge, mas adiciona custo e dependência externa
- DSL proprietário (`.prisma`) — curva de aprendizado inicial

**Segue com:**
- Schema em `prisma/schema.prisma`
- Migrations em `prisma/migrations/`
- Workflow de schema: ver `workflow/schema-prisma.md`
- Singleton em `lib/prisma.ts`

---

## Resumo

| Critério | Drizzle | Prisma |
|----------|---------|--------|
| Edge runtime | ✅ | ⚠️ (com Accelerate) |
| DX / tooling | ⚠️ | ✅ |
| Sintaxe próxima de SQL | ✅ | ❌ |
| Bundle size | ✅ | ⚠️ |
| Postgres complexo | ✅ | ✅ |
| SQLite / D1 / Turso | ✅ | ❌ |

**Regra rápida:** Cloudflare Workers → Drizzle. Node.js + Postgres → Prisma ou Drizzle (ambos funcionam bem).
