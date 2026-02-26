# Decisão: Real-time

Define como atualizações em tempo real chegam ao cliente sem polling.

## Quando você precisa de real-time

- Notificações ao vivo (nova mensagem, status de pagamento)
- Dashboards com dados atualizados automaticamente
- Colaboração simultânea (múltiplos usuários editando)
- Status de job em background (progresso de upload, geração de PDF)

> Antes de adicionar real-time, avaliar se **polling simples** (`setInterval` a cada 5s) resolve. Para a maioria dos casos de "atualização frequente", polling é suficiente e muito mais simples.

---

## Opções

### Server-Sent Events (SSE)

**Escolha quando:**
- Atualizações são **unidirecionais** (servidor → cliente)
- Runtime suporta streaming (Cloudflare Workers, Node.js, Vercel)
- Quer a solução mais simples sem biblioteca extra

**Trade-offs:**
- Apenas servidor → cliente — cliente não envia dados pelo canal SSE
- Reconexão automática nativa do browser, mas sem suporte a mensagens binárias
- Limitação de conexões simultâneas por browser (~6 por domínio em HTTP/1.1; sem limite em HTTP/2)

**Segue com:**
```typescript
// React Router / Hono — loader de SSE
export async function loader({ request }: LoaderArgs) {
  const stream = new ReadableStream({
    start(controller) {
      const send = (data: object) =>
        controller.enqueue(`data: ${JSON.stringify(data)}\n\n`);

      const interval = setInterval(() => send({ status: getJobStatus() }), 1000);
      request.signal.addEventListener("abort", () => {
        clearInterval(interval);
        controller.close();
      });
    },
  });
  return new Response(stream, {
    headers: { "Content-Type": "text/event-stream", "Cache-Control": "no-cache" },
  });
}

// Cliente
const source = new EventSource("/api/job-status");
source.onmessage = (e) => setStatus(JSON.parse(e.data));
```

---

### Supabase Realtime

**Escolha quando:**
- Banco já é Supabase
- Quer escutar mudanças diretamente no banco (Postgres CDC) sem escrever lógica de pub/sub
- Caso de uso: feed de atividade, notificações, sincronização de estado

**Trade-offs:**
- Acoplado ao Supabase
- Cada mudança na tabela vira um evento — pode gerar volume alto em tabelas com escritas frequentes
- Configurar RLS no realtime é obrigatório para segurança

**Segue com:**
```typescript
const channel = supabase
  .channel("invoices")
  .on("postgres_changes", { event: "INSERT", schema: "public", table: "invoices" },
    (payload) => setInvoices(prev => [...prev, payload.new])
  )
  .subscribe();
```

---

### PartyKit

**Escolha quando:**
- Colaboração em tempo real (múltiplos usuários editando o mesmo documento, whiteboard, cursor compartilhado)
- Runtime é Cloudflare — PartyKit roda em Durable Objects
- Quer WebSocket com estado compartilhado persistido na edge

**Trade-offs:**
- Overkill para notificações simples — use SSE nesses casos
- Modelo de programação de Durable Objects tem curva de aprendizado
- Custo proporcional ao número de "rooms" ativas

---

### Pusher / Ably

**Escolha quando:**
- Quer WebSocket bidirecional managed sem gerenciar infra
- Time não quer lidar com escala de conexões WebSocket

**Trade-offs:**
- Custo escala com conexões simultâneas e mensagens
- Vendor lock-in
- Pusher: popular mas interface antiga; Ably: mais moderno, mais caro

---

### WebSocket nativo

**Escolha quando:**
- Runtime é Railway / Fly.io / servidor persistente
- Precisa de controle total sobre o protocolo (binário, subprotocolos)
- Volume alto onde serviços managed seriam caros

**Trade-offs:**
- Você gerencia reconexão, rooms, broadcast, escala horizontal (com Redis pub/sub)
- Não funciona em serverless / edge sem adaptador

---

## Resumo

| Critério | SSE | Supabase RT | PartyKit | Pusher/Ably | WS nativo |
|----------|-----|-------------|----------|-------------|-----------|
| Bidirecional | ❌ | ✅ | ✅ | ✅ | ✅ |
| Edge / Cloudflare | ✅ | ❌ | ✅ | ✅ | ❌ |
| Serverless | ✅ | ✅ | ✅ | ✅ | ❌ |
| Colaboração | ❌ | ⚠️ | ✅ | ✅ | ✅ |
| Complexidade | ✅ | ✅ | ⚠️ | ✅ | ❌ |
| Custo | ✅ | ⚠️ | ⚠️ | ⚠️ | ✅ |

**Regra rápida:** notificações simples servidor → cliente → SSE. Já usa Supabase → Supabase Realtime. Colaboração / whiteboard → PartyKit. Bidirecional sem infra → Pusher/Ably. Servidor persistente com controle total → WS nativo.
