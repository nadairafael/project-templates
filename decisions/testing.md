# Decisão: Testing

Define a estratégia de testes. Impacta velocidade de feedback, confiança no deploy e CI.

## Camadas de teste

Decidir quais camadas fazem sentido para o projeto antes de escolher ferramentas:

| Camada | O que cobre | Quando vale |
|--------|-------------|-------------|
| **Unit** | Funções puras, utils, schemas Zod | Sempre |
| **Integration** | Repositories, handlers com DB real/mock | Projetos com lógica de negócio complexa |
| **E2E** | Fluxos críticos no browser | MVP+ com fluxos de pagamento, auth, onboarding |
| **Component** | Componentes React isolados | Design systems, componentes com lógica |

---

## Opções: Unit e Integration

### Vitest

**Escolha quando:**
- Stack usa Vite (React Router v7, TanStack Start, Vite puro)
- Quer cobertura de código nativa sem config extra
- Prioriza velocidade — paralelo e nativo ESM

**Trade-offs:**
- Ecossistema menor que Jest — alguns mocks mais trabalhosos
- Configuração de aliases precisa espelhar o `vite.config.ts`

**Segue com:**
```bash
bun add -D vitest @vitest/coverage-v8
```
```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
export default defineConfig({
  test: { environment: "node", coverage: { provider: "v8" } },
});
```

---

### Jest

**Escolha quando:**
- Projeto usa Next.js com App Router (suporte mais maduro)
- Dependências CJS pesadas que Vitest ainda não resolve bem
- Time já tem Jest configurado em outros projetos

**Trade-offs:**
- Mais lento que Vitest em repos grandes
- Config de ESM + path aliases exige mais setup

---

## Opções: E2E

### Playwright

**Escolha quando:**
- Precisa testar em múltiplos browsers (Chromium, Firefox, WebKit)
- Fluxos críticos: checkout, onboarding, auth
- CI é prioridade — Playwright é mais estável em headless

**Trade-offs:**
- Curva de aprendizado maior que Cypress
- Mais verboso para assertions simples

**Segue com:**
```bash
bun add -D @playwright/test
npx playwright install
```
```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";
export default defineConfig({
  testDir: "./e2e",
  use: { baseURL: "http://localhost:3000" },
});
```

---

### Cypress

**Escolha quando:**
- Time prefere DX interativa (Cypress Studio, time-travel debugging)
- Projeto é só Chromium — sem necessidade de cross-browser
- Componentes React precisam de teste visual interativo

**Trade-offs:**
- Mais lento em CI que Playwright
- Cross-browser exige plano pago
- Menos usado em projetos novos em 2025+

---

## Resumo

| Critério | Vitest | Jest | Playwright | Cypress |
|----------|--------|------|------------|---------|
| Velocidade | ✅ | ⚠️ | ✅ | ⚠️ |
| ESM nativo | ✅ | ⚠️ | — | — |
| Cross-browser | — | — | ✅ | ⚠️ (pago) |
| DX interativa | ⚠️ | ⚠️ | ⚠️ | ✅ |
| CI estabilidade | ✅ | ✅ | ✅ | ⚠️ |
| Next.js suporte | ⚠️ | ✅ | ✅ | ✅ |

**Regra rápida:** Vite-based → Vitest. Next.js → Jest ou Vitest. E2E → Playwright (padrão moderno).

## Stack de testing recomendada por projeto

| Tipo de projeto | Stack |
|----------------|-------|
| SaaS com React Router v7 / TanStack Start | Vitest + Playwright |
| SaaS com Next.js | Vitest (ou Jest) + Playwright |
| API pura / Worker | Vitest |
| MVP rápido | Vitest para utils críticos, sem E2E |
