# Decisão: Estilo de API

Define o contrato entre cliente e servidor. Impacta type-safety, flexibilidade e complexidade.

## Opções

### tRPC

**Escolha quando:**
- Stack é TypeScript puro full-stack (cliente e servidor no mesmo repo)
- Quer type-safety end-to-end sem geração de código
- Não há clientes externos (mobile nativo, terceiros) consumindo a API

**Trade-offs:**
- Acoplado ao TypeScript — clientes não-TS (mobile nativo, outros backends) não conseguem consumir diretamente
- Adiciona camada de abstração — debugging de rede é menos óbvio que REST
- Migrar para REST depois tem custo

**Segue com:**
```typescript
// server/router.ts
import { router, publicProcedure } from "./trpc";
import { z } from "zod";

export const appRouter = router({
  clients: router({
    list: publicProcedure.query(({ ctx }) => ctx.db.select().from(clients)),
    create: publicProcedure
      .input(z.object({ name: z.string() }))
      .mutation(({ input, ctx }) => ctx.db.insert(clients).values(input)),
  }),
});

export type AppRouter = typeof appRouter;
```

```typescript
// client
const { data } = trpc.clients.list.useQuery();
await trpc.clients.create.mutate({ name: "Acme" });
```

---

### REST

**Escolha quando:**
- API será consumida por clientes diversos (mobile nativo, terceiros, outros backends)
- Time já conhece REST e não quer curva de aprendizado
- Projeto usa React Router v7 / Next.js — loaders e actions já são "REST-like"

**Trade-offs:**
- Sem type-safety automática entre cliente e servidor — usar Zod + tipos compartilhados ou OpenAPI para compensar
- Mais boilerplate para CRUD simples

**Segue com:**
- Schemas Zod compartilhados entre cliente e servidor em `app/lib/schemas/`
- Tipagem de resposta via `typeof` nos endpoints ou geração de cliente via OpenAPI (Orval, Hey API)
- Estrutura de URL: `GET /api/clients`, `POST /api/clients`, `GET /api/clients/:id`

---

### GraphQL

**Escolha quando:**
- Dados são um grafo complexo com muitas relações e queries variáveis por cliente
- Múltiplos clientes com necessidades muito diferentes (mobile vs web vs parceiros)
- Time tem experiência com GraphQL

**Trade-offs:**
- Overhead significativo: schema, resolvers, N+1 problem, DataLoader
- Overkill para a maioria dos SaaS — REST ou tRPC resolvem com menos complexidade
- Cache HTTP não funciona out-of-the-box

**Segue com:**
- `graphql-yoga` ou Apollo Server
- Pothosuma para schema type-safe
- DataLoader obrigatório para evitar N+1

---

### Server Actions (Next.js / React Router)

**Escolha quando:**
- Framework suporta nativamente (Next.js App Router, React Router v7)
- Formulários e mutações simples — sem necessidade de API separada
- Quer progressive enhancement sem JavaScript

**Trade-offs:**
- Sem URL canônica — não serve para clientes externos
- Debugging mais difícil que REST explícito
- Limita portabilidade do código de servidor

---

## Resumo

| Critério | tRPC | REST | GraphQL | Server Actions |
|----------|------|------|---------|----------------|
| Type-safety automática | ✅ | ⚠️ (manual) | ✅ (com codegen) | ✅ |
| Clientes externos | ❌ | ✅ | ✅ | ❌ |
| Complexidade de setup | ⚠️ | ✅ | ❌ | ✅ |
| Flexibilidade de query | ⚠️ | ⚠️ | ✅ | ❌ |
| Cache HTTP | ❌ | ✅ | ❌ | ❌ |
| Curva de aprendizado | ⚠️ | ✅ | ❌ | ✅ |

**Regra rápida:** TypeScript full-stack sem clientes externos → tRPC. API pública ou mobile → REST. Dados em grafo complexo com múltiplos clientes → GraphQL. Forms simples no framework → Server Actions.
