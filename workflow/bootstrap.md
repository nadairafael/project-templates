# Checklist: Bootstrap de Projeto Novo

Processo para iniciar um projeto do zero usando este framework de documentacao. A ordem importa — cada fase se apoia na anterior.

---

## Fase 1: Definir o que

Objetivo: ter clareza sobre escopo antes de abrir o editor. Esse e o momento mais barato para mudar de ideia.

### 1.1 Escrever overview.md

Criar `docs/architecture/overview.md` com:

- [ ] **O que o sistema faz** — 2-3 frases. Se nao cabe em 3 frases, o escopo esta grande demais para um MVP
- [ ] **Casos de uso do MVP** — 3-5 casos de uso. Cada um em uma frase com ator + acao + resultado
- [ ] **Non-goals** — o que nao vai fazer. Ser explicito aqui evita scope creep. Exemplos:
  - "Nao tera app mobile nativo"
  - "Nao tera multi-tenancy"
  - "Nao tera API publica"
  - "Nao tera i18n"
- [ ] **NFRs basicos** — responder:
  - Precisa de autenticacao? Qual metodo?
  - Precisa de rate limiting?
  - Precisa ser responsivo? Mobile-first?
  - Qual idioma da interface?
  - Qual moeda/locale?

**Dica:** non-goals sao mais importantes que features nesse momento. Features voce vai descobrir. Non-goals voce precisa decidir.

### 1.2 Preencher project-context.md

Preencher `docs/workflow/project-context.md` com o contexto do produto — o que o agente precisa saber para trabalhar bem:

- [ ] **O que e o produto** — descricao em 3-5 frases (quem usa, qual problema resolve, como resolve)
- [ ] **Personas** — quem vai usar, o que espera
- [ ] **Fluxos principais** — sequencia de acoes do usuario
- [ ] **Mapa de rotas / telas** — rotas do MVP com descricao e se precisa de auth
- [ ] **Tom e personalidade** — como o produto se comunica (tom, mensagens de erro, empty states)
- [ ] **Nivel de refino visual** — fidelidade esperada, responsividade, animacoes, dark mode
- [ ] **Restricoes e premissas** — o que o agente precisa saber para nao errar
- [ ] **Fora de escopo (MVP)** — o que pode vir depois

Este documento complementa o overview.md: enquanto o overview define a arquitetura, o project-context define o produto.

### 1.3 Esbocar C4 (C1 e C2 apenas)

Criar `docs/architecture/c4.md` com:

- [ ] **C1 (Contexto)** — diagrama Mermaid com:
  - Quem usa o sistema (personas)
  - Quais sistemas externos interage (auth provider, APIs, bancos, email, etc.)
- [ ] **C2 (Containers)** — diagrama Mermaid com:
  - Quais artefatos deployaveis existem (SPA, API, banco, workers, etc.)
  - Como se comunicam entre si
  - Quais protocolos (HTTPS, WebSocket, TCP, etc.)

**Nao fazer C3 ainda.** Componentes internos so fazem sentido quando voce tiver codigo.

---

## Fase 2: Decidir como

Objetivo: tomar decisoes fundacionais e documenta-las ANTES de implementar. Decisoes tomadas aqui sao caras de mudar depois.

### 2.0 Consultar decisions/

Antes de criar ADRs, consultar os guias de decisao para cada area tecnologica. Cada arquivo explica as opcoes, trade-offs e o que segue de cada escolha.

Para cada decisao, voce tem duas saidas:
- **Escolher uma das opcoes listadas no guia** — o guia explica trade-offs e o que segue de cada escolha
- **"Ja sei o que quero"** — descrever livremente (lib, versao, motivo) e registrar como esta. Util para quem ja tem stack definida ou preferencias fortes

**Decisoes core — todo projeto precisa responder:**

| Decisao | Guia |
|---------|------|
| Runtime e deploy | `decisions/runtime.md` |
| Banco de dados | `decisions/database.md` |
| ORM | `decisions/orm.md` |
| Autenticacao | `decisions/auth.md` |
| Estilo de API (REST vs tRPC vs Server Actions) | `decisions/api-style.md` |
| UI e componentes | `decisions/ui.md` |
| Testing | `decisions/testing.md` |

**Decisoes condicionais — consultar se o projeto precisar:**

| Decisao | Guia |
|---------|------|
| Email transacional | `decisions/email.md` |
| Background jobs | `decisions/background-jobs.md` |
| File storage | `decisions/file-storage.md` |
| Real-time | `decisions/realtime.md` |
| Busca textual | `decisions/search.md` |
| Integracao com AI | `decisions/ai.md` |
| Error monitoring | `decisions/error-monitoring.md` |
| Monorepo | `decisions/monorepo.md` |

Registrar as decisoes tomadas em ADRs (proximo passo). Se a decisao foi direta (sem duvida entre alternativas), um comentario no `CLAUDE.md` do projeto e suficiente.

**Nota sobre UI:** ao decidir UI e componentes, coletar tambem referencias visuais (Figma, imagens, URLs, descricao textual). Ver secao "Referencias visuais" em `decisions/ui.md`.

### 2.1 ADRs fundacionais

Criar ADRs para cada decisao que seria cara de reverter. Usar `docs/architecture/adr/000-template.md`.

Decisoes tipicas de bootstrap (uma ADR por decisao fundacional):

- [ ] **Stack / Framework** — ver `decisions/runtime.md`
- [ ] **Banco de dados + ORM** — ver `decisions/database.md` + `decisions/orm.md`
- [ ] **Autenticacao** — ver `decisions/auth.md`
- [ ] **Estilo de API** — ver `decisions/api-style.md`
- [ ] **UI e componentes** — ver `decisions/ui.md`
- [ ] **Testing** — ver `decisions/testing.md`

Decisoes condicionais (ADR apenas se a decisao nao foi obvia):

- [ ] **Email** — ver `decisions/email.md`
- [ ] **Background jobs** — ver `decisions/background-jobs.md`
- [ ] **File storage** — ver `decisions/file-storage.md`
- [ ] **Real-time** — ver `decisions/realtime.md`
- [ ] **Busca textual** — ver `decisions/search.md`
- [ ] **AI** — ver `decisions/ai.md`
- [ ] **Monorepo** — ver `decisions/monorepo.md`
- [ ] **Campos monetarios** — Decimal vs Int (centavos) vs String?
- [ ] **Paginacao** — cursor vs offset vs carregar tudo?
- [ ] **Estado client** — loaders nativos vs TanStack Query vs Zustand?

**Regra:** se voce esta entre duas opcoes e nao consegue decidir em 5 minutos, crie o ADR com as duas alternativas. O exercicio de escrever pros/contras quase sempre revela a resposta.

### 2.2 Modelar dados

Criar `docs/data/data_dictionary.md` com:

- [ ] **Enums** — listar com valores e descricoes
- [ ] **Models** — para cada entidade do MVP:
  - Tabela: campo | tipo | nullable | default | unique | descricao
  - Relacoes (1:N, N:1, N:N)
  - Indices previstos
  - Constraints previstas
- [ ] **Diagrama de relacoes** — pode ser texto simples ou Mermaid ERD

**Nao precisa ser perfeito.** O schema vai evoluir. O objetivo e pensar nas entidades e relacoes antes de escrever `schema.prisma`, nao ter o modelo final.

Perguntas para guiar a modelagem:

- Quais sao as entidades (substantivos) do sistema?
- Quem pertence a quem? (relacoes)
- O que precisa ser unico? (constraints)
- Como vou consultar isso? (indices)
- O que acontece quando eu deletar X? (cascade)

---

## Fase 3: Configurar o projeto

Objetivo: scaffold tecnico funcional com as decisoes da fase 2 implementadas.

### 3.1 Buscar documentacao atualizada das libs

Antes de gerar qualquer arquivo, buscar a documentacao atual de cada lib do stack escolhido. Isso garante que comandos de instalacao, estrutura de pastas e configuracoes iniciais estejam corretos — nao baseados em versoes antigas.

- [ ] Para cada lib/framework relevante (framework, ORM, auth, UI): buscar docs via Context7 ou documentacao oficial
- [ ] Usar os docs como fonte primaria para os passos seguintes
- [ ] Se nao encontrar documentacao para alguma lib: usar conhecimento do agente e registrar nota no `CLAUDE.md` para revisar manualmente

### 3.2 Scaffold

- [ ] Inicializar projeto com framework escolhido (ex: `npx create-next-app@latest`)
- [ ] Instalar dependencias core (ORM, auth, styling)
- [ ] Configurar linter e formatter
- [ ] Configurar git + `.gitignore`

### 3.3 Schema inicial

- [ ] Escrever schema inicial baseado no data dictionary
- [ ] Seguir convencoes de `workflow/schema-drizzle.md` ou `workflow/schema-prisma.md` conforme ORM escolhido
- [ ] Rodar primeira migracao
- [ ] Se Prisma: rodar `prisma generate`

### 3.4 Infraestrutura de codigo

Criar esqueleto dos patterns que vao se repetir:

- [ ] **Auth** — helper de autenticacao (ex: `getAuthUser()`)
- [ ] **Error handling** — funcoes de resposta de erro padronizadas
- [ ] **Logger** — logging estruturado
- [ ] **Config** — arquivo de constantes centralizadas
- [ ] **Validacao** — funcoes de validacao de input
- [ ] **Rate limit** — se aplicavel

### 3.5 Adaptar docs de workflow

- [ ] `conventions.md` — preencher secoes `_definir_` com as decisoes tomadas na fase 2:
  - Idioma da UI e da documentacao
  - Estrutura de diretorios do scaffold
  - Biblioteca de forms escolhida
  - Estrategia de dados client-side
  - Breakpoint de responsividade
  - Scopes de commit do projeto
- [ ] `new-feature.md` — adaptar nomes de camadas ao stack escolhido
- [ ] Escolher o workflow de schema: `schema-drizzle.md` (Drizzle) ou `schema-prisma.md` (Prisma)

### 3.6 Criar CLAUDE.md

- [ ] Commands (dev, build, lint, migrate)
- [ ] Stack (uma linha)
- [ ] Convencoes iniciais (referenciar `docs/workflow/conventions.md`)
- [ ] Estrutura de diretorios

**Dica:** o CLAUDE.md comeca pequeno e cresce organicamente conforme padroes emergem. Nao tente escrever tudo no dia 1.

### 3.7 Primeiro commit

- [ ] Commit inicial com scaffold + docs + schema
- [ ] Mensagem: `chore: initial project setup with docs framework`

---

## Fase 4: Implementar MVP

Objetivo: construir os casos de uso definidos na fase 1, seguindo os processos definidos nas fases 2-3.

### 4.1 Priorizar

Ordenar os casos de uso do MVP por dependencia:

```
1. [caso de uso que nao depende de nada]
2. [caso de uso que depende do 1]
3. [caso de uso que depende do 1 e 2]
```

### 4.2 Para cada feature do MVP

- [ ] Seguir `workflow/new-feature.md` integralmente
- [ ] Se schema muda, seguir `workflow/schema-drizzle.md` ou `workflow/schema-prisma.md`
- [ ] Se decisao arquitetural nova surge, criar ADR (ou consultar `decisions/` se ja coberta)
- [ ] Atualizar `CLAUDE.md` conforme padroes emergem

### 4.3 Ao longo da implementacao

Coisas que voce vai descobrir durante o MVP e precisa capturar:

- [ ] Gotchas — bugs ou comportamentos inesperados do framework/lib. Anotar no CLAUDE.md
- [ ] Patterns — padroes que se repetem (ex: "todo hook usa AbortController"). Adicionar em `conventions.md`
- [ ] Decisoes taticas — escolhas menores que nao justificam ADR mas precisam ser lembradas. Adicionar no CLAUDE.md

---

## Fase 5: Consolidar

Objetivo: a documentacao reflete o que foi realmente construido, nao o que foi planejado.

### 5.1 Atualizar docs

- [ ] **`overview.md`** — reescrever casos de uso com o que realmente existe. Adicionar non-goals que surgiram durante o MVP
- [ ] **`c4.md`** — adicionar C3 (componentes internos agora existem). Adicionar fluxos de sequencia para os fluxos criticos
- [ ] **`data_dictionary.md`** — atualizar para refletir schema final do MVP. Adicionar sugestoes de indices e constraints
- [ ] **`conventions.md`** — incorporar padroes que emergiram
- [ ] **`CLAUDE.md`** — consolidar tudo que foi adicionado ao longo do MVP

### 5.2 Regenerar artefatos

- [ ] Se Prisma: `prisma generate` — DBML e ERD atualizados
- [ ] Se Drizzle: `drizzle-kit generate` — verificar migrations pendentes
- [ ] Verificar que ERD / schema reflete o estado atual do banco

### 5.3 Revisar ADRs

- [ ] Alguma decisao da fase 2 mudou durante a implementacao? Atualizar status (Substituida por ADR-XXX)
- [ ] Alguma decisao nova foi tomada sem ADR? Criar retroativamente

### 5.4 Atualizar README.md de docs

- [ ] `docs/README.md` — refletir estrutura final, ajustar ordem de leitura

---

## Resumo visual

```
Fase 1: DEFINIR          Fase 2: DECIDIR                Fase 3: CONFIGURAR
─────────────────         ──────────────────────         ─────────────────
overview.md               decisions/ (consultar)         Buscar docs via Context7
├─ O que faz              ├─ runtime, database, orm       Scaffold do projeto
├─ Casos de uso MVP       ├─ auth, api-style, ui          ├─ Framework + deps
├─ Non-goals              │   └─ refs visuais aqui        ├─ Schema inicial
└─ NFRs                   ├─ testing                      ├─ Auth, logger, config
                          └─ email, jobs, storage...      ├─ Preencher conventions.md
c4.md (C1 + C2)           ADRs fundacionais              ├─ CLAUDE.md
                          data_dictionary.md             └─ Primeiro commit

         │                         │                         │
         ▼                         ▼                         ▼

Fase 4: IMPLEMENTAR                         Fase 5: CONSOLIDAR
─────────────────                           ─────────────────
Para cada feature:                          Atualizar todos os docs
├─ new-feature.md checklist                 ├─ overview.md (real vs planejado)
├─ schema-drizzle.md ou schema-prisma.md    ├─ c4.md (adicionar C3 + fluxos)
├─ ADR ou decisions/ (se decisao nova)      ├─ data_dictionary.md (schema final)
└─ CLAUDE.md (gotchas, patterns)            ├─ conventions.md (patterns emergentes)
                                            └─ Revisar ADRs
```

---

## Erros comuns no bootstrap

| Erro | Consequencia | Como evitar |
|------|-------------|-------------|
| Pular non-goals | Scope creep — feature que ninguem pediu consome semanas | Escrever non-goals ANTES de features |
| Escolher stack sem ADR | 3 meses depois, ninguem lembra por que usa X | ADR para toda decisao fundacional |
| Modelar banco na hora de codar | Migracoes constantes, schema inconsistente | Data dictionary antes de schema.prisma |
| Nao definir convencoes cedo | Cada arquivo segue estilo diferente | conventions.md na fase 3, antes do primeiro feature |
| Over-engineer no dia 1 | Abstrair antes de ter 3 exemplos | Regra: nao abstrair ate ter 3 instancias concretas |
| Documentar demais no dia 1 | Docs ficam obsoletos antes do MVP | Docs minimos nas fases 1-3, consolidar na fase 5 |
| Documentar de menos | Decisoes perdidas, padroes inconsistentes | Capturar gotchas e patterns no CLAUDE.md durante a fase 4 |
| Copiar convencoes sem adaptar | Regras que nao se aplicam ao novo projeto | Revisar cada secao de conventions.md na fase 3 |
