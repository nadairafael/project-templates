# Decisão: AI Integration

Define como modelos de linguagem são integrados ao produto. Impacta latência, custo e DX.

## Opções

### Vercel AI SDK

**Escolha quando:**
- Stack é React (Next.js, React Router, TanStack Start)
- Precisa de streaming de respostas com UI reativa
- Quer suporte a múltiplos providers (Anthropic, OpenAI, Google, Groq) com mesma API
- Structured output (JSON tipado) é necessário

**Trade-offs:**
- Abstração adicional — debugging pode ser menos direto que SDK nativo
- Algumas features específicas de provider não estão disponíveis via AI SDK

**Segue com:**
```typescript
// Server (loader / action / route handler)
import { streamText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

export async function action({ request }: ActionArgs) {
  const { messages } = await request.json();

  const result = await streamText({
    model: anthropic("claude-sonnet-4-6"),
    system: "Você é um assistente financeiro.",
    messages,
  });

  return result.toDataStreamResponse();
}

// Client
import { useChat } from "@ai-sdk/react";

export function Chat() {
  const { messages, input, handleSubmit, handleInputChange } = useChat({
    api: "/api/chat",
  });
  // ...
}
```

```typescript
// Structured output
import { generateObject } from "ai";
import { z } from "zod";

const { object } = await generateObject({
  model: anthropic("claude-sonnet-4-6"),
  schema: z.object({
    items: z.array(z.object({ name: z.string(), amount: z.number() })),
  }),
  prompt: "Extraia os itens desta nota fiscal: ...",
});
```

---

### SDK direto do provider

**Escolha quando:**
- Usa apenas um provider e quer acesso a todas as features (tool use, cache de contexto, vision)
- Sem necessidade de streaming com hooks React
- Backend puro (worker, job, script)

**Segue com:**
```typescript
// Anthropic
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic({ apiKey: env.ANTHROPIC_API_KEY });

const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Analise este contrato..." }],
});

// Com cache de contexto (reduz custo em prompts longos reutilizados)
const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  system: [{ type: "text", text: longSystemPrompt, cache_control: { type: "ephemeral" } }],
  messages,
});
```

---

### LangChain / LlamaIndex

**Escolha quando:**
- Pipelines complexos: RAG com múltiplas fontes, agentes com ferramentas, memória persistida
- Prototipação rápida de pipelines antes de implementar manualmente

**Trade-offs:**
- Heavy — adiciona muita abstração e dependências
- Debugging difícil — erros internos às vezes são obscuros
- Para produção, considerar reimplementar o pipeline manualmente após prototipar

---

## Modelos recomendados por caso de uso (2025)

| Caso de uso | Modelo recomendado |
|-------------|-------------------|
| Chat / assistente geral | `claude-sonnet-4-6` (Anthropic) |
| Raciocínio complexo, código | `claude-opus-4-6` (Anthropic) |
| Tarefas rápidas / classificação | `claude-haiku-4-5` (Anthropic) |
| Geração de imagem | `dall-e-3` (OpenAI) ou `imagen-3` (Google) |
| Embedding / busca semântica | `text-embedding-3-small` (OpenAI) |
| Multimodal (visão) | `claude-sonnet-4-6` ou `gpt-4o` |
| Velocidade máxima / custo mínimo | `groq/llama-3.1-8b-instant` |

---

## Boas práticas

| Prática | Motivo |
|---------|--------|
| Nunca expor API keys no cliente | Toda chamada à API de AI passa pelo servidor |
| Implementar rate limiting por usuário | Custo por token — um usuário pode drenar o budget |
| Logar inputs e outputs (com opt-in do usuário) | Debug e custo tracking |
| Cache de respostas determinísticas | Mesmo input → mesmo output: cachear por hash do prompt |
| Structured output com Zod | Evitar parsear JSON manual — usar `generateObject` do AI SDK |
| Streaming para respostas longas | UX muito melhor que esperar a resposta completa |

---

## Resumo

| Critério | Vercel AI SDK | SDK direto | LangChain |
|----------|---------------|------------|-----------|
| Multi-provider | ✅ | ❌ | ✅ |
| Streaming com React | ✅ | ⚠️ (manual) | ⚠️ |
| Structured output | ✅ | ✅ | ✅ |
| Features específicas de provider | ⚠️ | ✅ | ⚠️ |
| Complexidade de pipeline | ⚠️ | ⚠️ | ✅ |
| Bundle size | ⚠️ | ✅ | ❌ |

**Regra rápida:** app React com chat / streaming → Vercel AI SDK. Worker / job / backend puro → SDK direto do provider. RAG ou agentes complexos → prototipar com LangChain, reimplementar manualmente para produção.
