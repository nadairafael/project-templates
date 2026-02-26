# Decisão: Error Monitoring

Define como erros de produção são capturados, alertados e investigados.

## Opções

### Sentry

**Escolha quando:**
- Quer solução completa: erros, performance, session replay, alertas
- Time precisa de stack traces com source maps, breadcrumbs e contexto de usuário
- Projeto tem budget — Sentry fica caro acima do free tier (50k erros/mês)

**Trade-offs:**
- Custo escala rápido — planos pagos a partir de $26/mês
- SDK adiciona ~20-30kb ao bundle client-side
- Edge Workers: suporte existe mas com algumas limitações

**Segue com:**
```typescript
// app/root.tsx ou entry point
import * as Sentry from "@sentry/react";
Sentry.init({ dsn: env.SENTRY_DSN, environment: env.NODE_ENV });

// Capturar erro manual
Sentry.captureException(error, { extra: { userId, context } });
```

---

### Axiom

**Escolha quando:**
- Runtime é Cloudflare Workers (melhor suporte que Sentry para edge)
- Quer logs estruturados + traces em vez de apenas erros
- Precisa de queries customizadas em logs (Axiom usa APL, linguagem própria)
- Budget menor — free tier mais generoso que Sentry

**Trade-offs:**
- Não é monitoramento de erros clássico — é mais observabilidade / logging
- Sem session replay
- Curva de aprendizado na linguagem de query (APL)

**Segue com:**
```typescript
import { Axiom } from "@axiomhq/js";
const axiom = new Axiom({ token: env.AXIOM_TOKEN });

// Log estruturado
axiom.ingest("logs", [{
  level: "error",
  message: error.message,
  stack: error.stack,
  userId,
  route: request.url,
}]);
await axiom.flush();
```

---

### Highlight.io

**Escolha quando:**
- Quer session replay integrado com erros (ver exatamente o que o usuário fez antes do erro)
- Open source é requisito (self-host disponível)
- Quer alternativa mais barata ao Sentry com DX similar

**Trade-offs:**
- Ecossistema menor — menos integrações que Sentry
- Self-host requer infra significativa (ClickHouse, Redis, etc.)

---

### LogSnag

**Escolha quando:**
- Quer notificações de eventos de negócio, não apenas erros técnicos
- Casos: "usuário fez upgrade", "pagamento falhou", "novo cadastro"
- Complemento a outro sistema de monitoring — não substitui Sentry/Axiom

**Trade-offs:**
- Não é error monitoring — é event tracking / notificações
- Não captura stack traces nem contexto técnico

---

## Resumo

| Critério | Sentry | Axiom | Highlight | LogSnag |
|----------|--------|-------|-----------|---------|
| Stack traces + source maps | ✅ | ⚠️ | ✅ | ❌ |
| Session replay | ✅ | ❌ | ✅ | ❌ |
| Edge / Cloudflare | ⚠️ | ✅ | ⚠️ | ✅ |
| Logs estruturados | ⚠️ | ✅ | ⚠️ | ❌ |
| Self-host | ✅ | ❌ | ✅ | ❌ |
| Custo (free tier) | ⚠️ | ✅ | ✅ | ✅ |

**Regra rápida:** Node.js / Vercel → Sentry (padrão de mercado). Cloudflare Workers → Axiom. Quer session replay sem Sentry → Highlight. Notificações de eventos de negócio → LogSnag como complemento.
