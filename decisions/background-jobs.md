# Decisão: Background Jobs

Define como tarefas assíncronas e agendadas são executadas. Impacta confiabilidade, observabilidade e infra.

## Quando você precisa de background jobs

- Envio de e-mail / WhatsApp após ação do usuário
- Geração de PDF, relatório ou exportação
- Sincronização com APIs externas
- Cobrança recorrente (Stripe webhooks, renovação de subscriptions)
- Tarefas agendadas (cron): lembretes, relatórios diários, cleanup

---

## Opções

### Trigger.dev v3

**Escolha quando:**
- Runtime é Node.js (Vercel, Railway, Fly.io)
- Quer observabilidade nativa: logs por task, replay de falhas, histórico
- Tasks complexas com steps, retries e timeouts configuráveis
- Time prefere DX moderna com TypeScript puro

**Trade-offs:**
- Requer conta Trigger.dev (self-host disponível mas com mais config)
- Não funciona em Cloudflare Workers (executa em Node.js)
- Free tier limitado para volume alto

**Segue com:**
```typescript
// trigger/send-invoice.ts
import { task } from "@trigger.dev/sdk/v3";

export const sendInvoiceEmail = task({
  id: "send-invoice-email",
  retry: { maxAttempts: 3, backoff: { type: "exponential" } },
  run: async (payload: { invoiceId: string }) => {
    const invoice = await getInvoice(payload.invoiceId);
    await resend.emails.send({ ... });
  },
});

// Disparar de uma action / loader:
await sendInvoiceEmail.trigger({ invoiceId });
```

---

### Inngest

**Escolha quando:**
- Quer modelo event-driven (funções reagem a eventos, não são chamadas diretamente)
- Workflows complexos com steps paralelos e condicionais
- Runtime é Vercel, Railway ou qualquer Node.js

**Trade-offs:**
- Mental model diferente de job queue tradicional — curva de aprendizado
- Requer conta Inngest (self-host mais complexo que Trigger.dev)

**Segue com:**
```typescript
// inngest/send-invoice.ts
export const sendInvoice = inngest.createFunction(
  { id: "send-invoice", retries: 3 },
  { event: "invoice/created" },
  async ({ event, step }) => {
    const invoice = await step.run("fetch-invoice", () => getInvoice(event.data.invoiceId));
    await step.run("send-email", () => resend.emails.send({ ... }));
  }
);

// Disparar:
await inngest.send({ name: "invoice/created", data: { invoiceId } });
```

---

### BullMQ

**Escolha quando:**
- Redis já está na infra (Railway, Upstash)
- Precisa de filas com prioridade, concorrência controlada e rate limiting
- Volume alto de jobs — BullMQ escala bem
- Quer solução self-hosted sem dependência de serviço externo

**Trade-offs:**
- Requer Redis — mais infra para gerenciar
- Observabilidade manual (Bull Board para UI, mas é básico)
- Mais boilerplate que Trigger.dev ou Inngest

---

### Cloudflare Queues

**Escolha quando:**
- Runtime é Cloudflare Workers
- Jobs simples: enviar mensagem, processar webhook, acionar outro worker
- Quer zero infra extra — tudo dentro do ecossistema Cloudflare

**Trade-offs:**
- Só funciona dentro do Cloudflare — sem portabilidade
- Sem steps, sem retry configurável por job, sem observabilidade rica
- Timeout de execução do Worker ainda se aplica (CPU limit)

---

### Cron nativo do framework / plataforma

**Escolha quando:**
- Tarefas periódicas simples (sem fila, sem retry elaborado)
- Cloudflare: `Cron Triggers` no `wrangler.jsonc`
- Vercel: `vercel.json` com `crons`
- Railway: `railway.toml` com scheduler

**Trade-offs:**
- Sem retry em caso de falha — implementar manualmente
- Sem histórico de execução out-of-the-box

---

## Resumo

| Critério | Trigger.dev | Inngest | BullMQ | CF Queues | Cron nativo |
|----------|-------------|---------|--------|-----------|-------------|
| Observabilidade | ✅ | ✅ | ⚠️ | ❌ | ❌ |
| Cloudflare Workers | ❌ | ❌ | ❌ | ✅ | ✅ |
| Self-hosted | ⚠️ | ⚠️ | ✅ | ✅ | ✅ |
| Steps / workflows | ✅ | ✅ | ⚠️ | ❌ | ❌ |
| Infra extra | ❌ | ❌ | Redis | ❌ | ❌ |
| DX / setup | ✅ | ✅ | ⚠️ | ⚠️ | ✅ |

**Regra rápida:** Node.js (Vercel/Railway) → Trigger.dev. Cloudflare Workers → CF Queues + Cron Triggers. Redis já na infra → BullMQ. Workflows event-driven complexos → Inngest.
