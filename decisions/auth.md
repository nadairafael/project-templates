# Decisão: Autenticação

Define como usuários se autenticam. Impacta multi-tenancy, providers OAuth, schema e custos.

## Opções

### better-auth

**Escolha quando:**
- Projeto precisa de organizações / multi-tenancy nativo (convites, roles, org switcher)
- Self-hosted é requisito — dados de usuário não saem da sua infra
- Runtime é edge (Cloudflare Workers) ou Node.js
- Providers: Google, GitHub, e-mail/senha, magic link, passkey

**Trade-offs:**
- Mais configuração inicial que Clerk
- UI de login é responsabilidade da equipe (sem componentes prontos)
- Ecossistema menor, documentação ainda crescendo
- Schema de auth gerado automaticamente — não editar manualmente

**Segue com:**
- `lib/auth/auth.server.ts` — configuração do servidor
- `lib/auth/auth-client.ts` — cliente React
- Schema auto-gerado em `database/auth-schema.ts` (Drizzle) ou `prisma/schema.prisma` (Prisma)
- Rota `/api/auth/*` como handler catch-all
- Middleware para injetar `session` e `activeOrganizationId` em rotas privadas

---

### Clerk

**Escolha quando:**
- Speed-to-market é prioridade máxima
- Precisa de UI de auth pronta (login, signup, perfil, org switcher — componentes React incluídos)
- Budget existe para serviço pago ($25+/mês para features avançadas)
- Organizações e invitations são features core e você não quer implementar

**Trade-offs:**
- Vendor lock-in — migrar depois é caro (dados de usuário ficam no Clerk)
- Custo escala com MAU (free até 10k usuários)
- Sincronização via webhook necessária para manter dados de usuário no seu banco

**Segue com:**
- `@clerk/nextjs` ou `@clerk/react-router` dependendo do framework
- Sem schema de auth próprio — Clerk gerencia tudo
- Webhook `/api/webhooks/clerk` para sync de user → seu banco
- `clerkMiddleware()` no middleware principal

---

### Auth.js (NextAuth v5)

**Escolha quando:**
- Projeto é Next.js e multi-tenancy não é requisito
- Self-hosted sem precisar de organizações nativo
- Comunidade e volume de exemplos são prioridade

**Trade-offs:**
- Multi-tenancy não é nativo — implementar organizações manualmente
- v5 ainda em evolução — alguns adapters instáveis
- Menos opinionated que better-auth (mais liberdade, mais decisões para tomar)

**Segue com:**
- `auth.ts` na raiz do projeto
- `app/api/auth/[...nextauth]/route.ts`
- Adapter Prisma ou Drizzle para persistir sessões

---

### Supabase Auth

**Escolha quando:**
- Banco já é Supabase e quer Auth integrado com Row Level Security (RLS)
- Quer aproveitar Supabase Realtime + Auth no mesmo lugar

**Trade-offs:**
- Acoplado ao Supabase — trocar de banco implica trocar de auth
- RLS exige conhecimento de SQL policies para funcionar corretamente

---

## Resumo

| Critério | better-auth | Clerk | Auth.js | Supabase |
|----------|-------------|-------|---------|----------|
| Multi-tenancy nativo | ✅ | ✅ | ❌ | ❌ |
| Self-hosted | ✅ | ❌ | ✅ | ❌ |
| UI de auth pronta | ❌ | ✅ | ❌ | ⚠️ |
| Edge compatible | ✅ | ✅ | ⚠️ | ❌ |
| Custo | Grátis | Freemium | Grátis | Freemium |
| Complexidade de setup | Média | Baixa | Baixa | Baixa |

**Regra rápida:** multi-tenancy + self-hosted → better-auth. Velocidade máxima → Clerk. Next.js simples sem org → Auth.js. Já usa Supabase → Supabase Auth.
