# Decisão: Runtime e Deploy

Define onde o código roda. Impacta escolha de banco, ORM e limitações de API disponíveis.

## Opções

### Cloudflare Workers + Pages

**Escolha quando:**
- App precisa de latência baixa globalmente (edge em 300+ PoPs)
- Não há dependência de APIs Node.js nativas (`fs`, `child_process`, streams)
- Banco é D1, Turso ou Postgres externo (Neon, Supabase)
- Custo de compute é sensível (free tier generoso, billing por request)

**Trade-offs:**
- Sem Node.js nativo — algumas libs não funcionam out-of-the-box
- Limite de CPU por request (não serve para jobs pesados ou processamento longo)
- Debug local via `wrangler dev` — comportamento levemente diferente de produção
- Variáveis de ambiente gerenciadas via `wrangler.jsonc` + secrets

**Segue com:**
- ORM: Drizzle (obrigatório para D1; Prisma não suporta D1 nativamente)
- Banco: D1 (SQLite) ou Postgres externo
- Config: `wrangler.jsonc` para bindings (D1, KV, R2, secrets)
- Deploy: `wrangler deploy`
- Framework recomendado: React Router v7, Hono, ou Remix

---

### Vercel

**Escolha quando:**
- Projeto é Next.js (integração oficial e mais madura)
- Equipe quer deploy zero-config com preview environments por PR
- Banco é Neon ou Supabase (integração nativa no dashboard)
- Edge Middleware é suficiente — sem necessidade de servidor persistente

**Trade-offs:**
- Cold start em serverless functions (mitigado com Fluid Compute no plano Pro)
- Free tier limitado para produção com tráfego real
- Custo sobe rápido com muitas serverless invocations

**Segue com:**
- ORM: Prisma ou Drizzle (ambos funcionam)
- Banco: Neon (serverless Postgres com branching) ou Supabase
- Framework: Next.js (primeira opção), React Router v7 (funciona com adaptador)

---

### Railway

**Escolha quando:**
- App precisa de servidor persistente (WebSockets, background workers, cron jobs)
- Dependências de Node.js nativas são necessárias
- Precisa de Redis, Postgres e app no mesmo lugar com rede privada
- Equipe prefere modelo container / servidor tradicional

**Trade-offs:**
- Custo fixo mesmo com tráfego baixo (mínimo ~$5/mês)
- Sem edge — latência depende da região escolhida (us-west, eu-west, etc.)
- Mais operacional que Vercel/Cloudflare para projetos simples

**Segue com:**
- ORM: Prisma (melhor DX com Node.js + Postgres)
- Banco: Postgres nativo do Railway
- Framework: qualquer (Express, Fastify, Next.js, Remix)
- Jobs: background workers como serviço separado no mesmo projeto

---

## Resumo

| Critério | Cloudflare | Vercel | Railway |
|----------|-----------|--------|---------|
| Latência global (edge) | ✅ | ✅ | ❌ |
| Node.js nativo | ❌ | ✅ | ✅ |
| Servidor persistente / WebSocket | ❌ | ❌ | ✅ |
| Free tier generoso | ✅ | ⚠️ | ❌ |
| Preview por PR | ✅ | ✅ | ⚠️ |
| SQLite / D1 | ✅ | ❌ | ❌ |
| Postgres gerenciado | ❌ (externo) | ✅ (Neon) | ✅ |

**Regra rápida:** app global read-heavy sem Node nativo → Cloudflare. SaaS Next.js padrão → Vercel. Precisa de WebSocket, cron ou Redis → Railway.
