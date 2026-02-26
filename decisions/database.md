# Decisão: Banco de dados

Escolha o banco antes do ORM — alguns ORMs não suportam determinados bancos.

## Opções

### Cloudflare D1 (SQLite na edge)

**Escolha quando:**
- Runtime é Cloudflare Workers
- Modelo de dados é relativamente simples (sem queries analíticas complexas)
- Leituras são muito mais frequentes que escritas
- Custo é prioridade (free tier: 5GB storage, 10M rows lidas/dia)

**Trade-offs:**
- SQLite: sem alguns tipos nativos (`BOOLEAN` vira `INTEGER 0/1`, sem `ARRAY`)
- Sem transações distribuídas entre múltiplos workers
- Writes replicam com pequena latência — não use D1 para contadores de alta frequência
- ORM obrigatório: Drizzle (Prisma não suporta D1)

---

### PostgreSQL

**Escolha quando:**
- Modelo de dados é complexo (muitas relações, queries analíticas, JSONB)
- Precisa de features SQL avançadas (CTEs, window functions, full-text search, `pg_vector`)
- App tem writes frequentes e concorrência alta
- Runtime é Node.js ou Vercel

**Provedores:**

| Provedor | Quando usar |
|----------|-------------|
| **Neon** | Serverless Postgres com branching por branch de git — ótimo para times |
| **Supabase** | Precisa de Auth + Storage + Realtime junto com Postgres no mesmo lugar |
| **Railway** | Postgres simples gerenciado, junto com o app na mesma rede privada |
| **PlanetScale** | MySQL-compatible com sharding e deploy branching (schema changes sem lock) |

**Trade-offs:**
- Custo maior que D1 para projetos com tráfego baixo
- Requer connection pooling em ambientes serverless (PgBouncer, Neon serverless driver)

---

### Turso (SQLite distribuído)

**Escolha quando:**
- Quer SQLite mas não está no Cloudflare (ex: Bun + Hono, Fly.io)
- App é multi-region e quer latência baixa de leitura por região
- Modelo de dados é simples e não requer Postgres

**Trade-offs:**
- Menos maduro que Postgres em ecossistema e tooling
- Vendor-specific — migrar é mais trabalhoso que trocar provider de Postgres

---

## Resumo

| Critério | D1 | PostgreSQL | Turso |
|----------|----|------------|-------|
| Edge nativo | ✅ | ❌ | ✅ |
| SQL completo | ⚠️ | ✅ | ⚠️ |
| Custo baixo (baixo tráfego) | ✅ | ⚠️ | ✅ |
| Escala de writes | ⚠️ | ✅ | ⚠️ |
| Tooling / ecossistema | ✅ | ✅ | ⚠️ |
| ORM compatível | Drizzle | Drizzle, Prisma | Drizzle |

**Regra rápida:** Cloudflare Workers → D1. Node.js + queries complexas → Postgres (Neon ou Supabase). Bun multi-region sem Cloudflare → Turso.
