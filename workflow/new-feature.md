# Checklist: Nova Feature

Processo para adicionar funcionalidade nova ao projeto — da ideia ao deploy.

---

## 0. Antes de comecar

- [ ] A feature esta dentro do escopo? Consultar **non-goals** em `docs/architecture/overview.md`
- [ ] Existe ADR relevante que impacta essa feature? Consultar `docs/architecture/adr/`
- [ ] A feature exige decisao arquitetural nova? Se sim, criar ADR antes de implementar (usar `adr/000-template.md`)

---

## 1. Definir escopo

### 1.1 Descrever a feature

Responder em uma ou duas frases:

- **O que:** o que o usuario consegue fazer que nao conseguia antes?
- **Por que:** qual problema resolve ou qual valor entrega?
- **Onde:** em qual pagina/fluxo aparece?

### 1.2 Identificar canais

A feature afeta quais interfaces?

- [ ] **Web** — componentes React, hooks, API routes
- [ ] **Integracoes externas** — bots, webhooks, APIs de terceiros
- [ ] **Nenhum canal novo** — mudanca interna (refactor, performance, infra)

### 1.3 Listar criterios de aceite

O que precisa ser verdade para a feature estar completa?

```
- [ ] Usuario consegue [acao]
- [ ] Dados sao persistidos em [tabela]
- [ ] Funciona em mobile e desktop
- [ ] Mensagens no idioma definido
- [ ] ...
```

---

## 2. Analisar impacto

### 2.1 Mapear camadas afetadas

| Camada | Afetada? | O que muda |
|--------|----------|------------|
| **Schema** (prisma) | [ ] | Novo model? Novo campo? Novo enum? |
| **Servicos** (lib/services/) | [ ] | Nova funcao? Alterar existente? |
| **API routes** (app/api/) | [ ] | Nova rota? Alterar existente? |
| **Validators** (lib/api/validators.ts) | [ ] | Novas validacoes? |
| **Rate limit** (lib/config.ts) | [ ] | Nova constante de rate limit? |
| **Hooks** (lib/hooks/) | [ ] | Novo hook? Alterar existente? |
| **Componentes** (components/) | [ ] | Novo componente? Alterar existente? |
| **Types** (types/api.ts) | [ ] | Novas interfaces? |
| **CSS** (globals.css) | [ ] | Novas variaveis? Novas classes? |
| _Integracoes_ | [ ] | _Adaptar conforme o projeto_ |

### 2.2 Se schema muda

Seguir `docs/workflow/schema-change.md` integralmente antes de prosseguir.

### 2.3 Estimar complexidade

| Sinal | Complexidade |
|-------|-------------|
| Muda 1-2 arquivos, zero schema | Baixa — implementar direto |
| Muda 3-5 arquivos, schema simples (campo nullable) | Media — planejar ordem |
| Muda 6+ arquivos, schema breaking | Alta — criar ADR, dividir em PRs |

---

## 3. Implementar

Ordem bottom-up: dados → servicos → API → client. Cada camada se apoia na anterior.

### 3.1 Schema e migracao

_(Pular se nao afeta banco)_

- [ ] Editar `prisma/schema.prisma` seguindo convencoes (ver `schema-change.md`)
- [ ] `npx prisma migrate dev --name <nome>`
- [ ] Revisar SQL gerado
- [ ] Se campo Decimal novo, adicionar conversao em `lib/prisma.ts`
- [ ] `npx prisma generate`

### 3.2 Types

- [ ] Atualizar `types/api.ts` com interfaces de request/response da feature

### 3.3 Servico

_(Pular se logica cabe inteira na API route)_

- [ ] Criar ou alterar servico em `lib/services/`
- [ ] Se manipula multiplos registros, usar `prisma.$transaction({ timeout })`
- [ ] Se servico sera compartilhado entre canais, extrair funcao pura (sem dependencia de `Request`/`Response`)

### 3.4 API route

- [ ] Criar ou alterar rota em `app/api/`
- [ ] Autenticacao no inicio
- [ ] Rate limit com constante centralizada em `lib/config.ts`
- [ ] Validar input:
  - [ ] `try/catch` em `request.json()` → mensagem de erro
  - [ ] Validar campos obrigatorios
- [ ] Erros no idioma do projeto, sem detalhes internos
- [ ] Logger estruturado
- [ ] Queries scoped por `userId`

### 3.5 Integracoes externas

_(Pular se nao se aplica)_

- [ ] Adaptar esta secao conforme as integracoes do projeto (bots, webhooks, APIs de terceiros)

### 3.6 Hook client

- [ ] Criar ou alterar hook em `lib/hooks/`
- [ ] AbortController com cleanup no unmount
- [ ] `useCallback` para funcoes de fetch e handlers
- [ ] Se updates otimistas: estado local → API call → rollback on error

### 3.7 Componentes

- [ ] Criar ou alterar em `components/`
- [ ] Design system:
  - [ ] Cores: `var(--color-*)`, nunca hex hardcoded
  - [ ] Espacamento: `var(--space-*)`, nunca px hardcoded
  - [ ] Tipografia: presets de `lib/utils.ts`
  - [ ] Radius, sombras, opacidades: variaveis CSS
- [ ] Reutilizar componentes existentes antes de criar novos
- [ ] Textos no idioma do projeto

### 3.8 Acessibilidade

- [ ] Elementos interativos tem `focus-visible`
- [ ] Dialogs: `aria-modal`, `aria-labelledby`, focus trap, Escape fecha
- [ ] Estados desabilitados: `opacity`, `pointer-events: none`
- [ ] Animacoes respeitam `prefers-reduced-motion`

### 3.9 Responsividade

- [ ] Mobile-first
- [ ] Usar variaveis de layout
- [ ] Testar em viewport mobile e desktop
- [ ] Dialogs: `max-width: 100%` para nao estourar em mobile

---

## 4. Verificar

### 4.1 Build e lint

- [ ] `npx prisma generate` — sem erros
- [ ] `npm run build` — sem erros de tipo
- [ ] `npm run lint` — sem erros de lint

### 4.2 Teste manual

- [ ] Fluxo funciona end-to-end no browser
- [ ] Fluxo funciona em mobile
- [ ] Erros tratados: input invalido, rede offline, rate limit

### 4.3 Integridade de dados

- [ ] Transacoes atomicas: erro parcial nao deixa dados inconsistentes?
- [ ] Cascade delete: deletar entidade pai nao deixa orfaos?

---

## 5. Documentar

- [ ] **`CLAUDE.md`** — atualizar se nova convencao, novo padrao, nova rota, novo servico
- [ ] **`docs/data/data_dictionary.md`** — se schema mudou
- [ ] **`docs/architecture/c4.md`** — se novo container, componente ou fluxo significativo
- [ ] **`docs/architecture/overview.md`** — se novo caso de uso ou novo non-goal
- [ ] **`docs/architecture/adr/`** — se decisao arquitetural nova (usar template 000)
- [ ] **`README.md`** — se afeta setup, env vars ou uso
- [ ] Regenerar DBML e ERD se schema mudou: `npx prisma generate`

---

## 6. Commit e deploy

- [ ] Conventional Commits: `feat(scope): descricao`
- [ ] Se feature grande, dividir em commits atomicos (schema → servico → API → UI)
- [ ] Documentacao atualizada no mesmo PR ou commit separado

---

## Exemplos por tipo de feature

### Feature simples: novo campo no formulario

```
Escopo: adicionar campo opcional ao dialog.
Camadas: schema (campo nullable) → types → API route → hook → componente
Complexidade: baixa
```

### Feature media: nova secao na pagina principal

```
Escopo: nova secao com dados agregados.
Camadas: schema (novo model) → servico → API route (CRUD)
         → hook → componente → documentacao
Complexidade: media
```

### Feature complexa: nova integracao externa

```
Escopo: integracao com servico externo.
Camadas: schema (IDs no User) → servico (compartilhar logica existente)
         → API route (webhook) → modulos de integracao
         → config UI (vincular/desvincular) → documentacao completa
Complexidade: alta — criar ADR primeiro
```

---

## Antipatterns a evitar

| Antipattern | Alternativa correta |
|-------------|-------------------|
| Hardcodar cor hex no componente | Usar `var(--color-*)` |
| Inline style com fontSize/fontWeight | Usar presets de tipografia |
| `console.error` em API route | Usar logger estruturado |
| `!important` em CSS | Aumentar especificidade do seletor |
| `useEffect` sem AbortController | Sempre incluir cleanup |
| `$transaction` sem timeout | Sempre incluir timeout |
| `request.json()` sem try/catch | Sempre wrap em try/catch |
| FK sem `@@index` | Toda FK nova recebe indice |
| Criar componente novo sem checar existentes | Reutilizar componentes existentes |
