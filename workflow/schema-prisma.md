# Checklist: Mudanca de Schema

Processo para qualquer alteracao no `prisma/schema.prisma` — novo model, novo campo, renomear enum, adicionar constraint, etc.

---

## 1. Analise de impacto

Antes de tocar no schema, mapear o que sera afetado.

### 1.1 Identificar dependentes

Para o model/campo que sera alterado, verificar:

- [ ] **API routes** — quais rotas fazem queries com esse campo?
- [ ] **Servicos** (`lib/services/`) — algum servico referencia esse campo?
- [ ] **Hooks client** (`lib/hooks/`) — hooks de fetch usam esse campo na tipagem ou logica?
- [ ] **Componentes** (`components/`) — algum componente renderiza ou filtra por esse campo?
- [ ] **Validators** (`lib/api/validators.ts`) — existe validacao para esse campo?
- [ ] **Types** (`types/api.ts`) — o tipo aparece em interfaces compartilhadas?
- [ ] **Integracoes** — _adaptar conforme o projeto (IA tools, bots, webhooks, etc.)_

### 1.2 Classificar o risco

| Tipo de mudanca | Risco | Requer migracao de dados? |
|-----------------|-------|--------------------------|
| Adicionar campo nullable com default | Baixo | Nao |
| Adicionar campo NOT NULL com default | Baixo | Nao |
| Adicionar campo NOT NULL sem default | **Alto** | Sim (precisa popular antes) |
| Adicionar novo model | Baixo | Nao |
| Adicionar novo enum value | Baixo | Nao |
| Renomear campo | **Alto** | Sim (downtime ou rename SQL) |
| Renomear enum value | Medio | Sim (`ALTER TYPE ... RENAME VALUE`) |
| Remover campo | Medio | Sim (verificar que nenhum codigo referencia) |
| Remover model | **Alto** | Sim (verificar cascades e FKs) |
| Mudar tipo de campo | **Alto** | Sim (conversao de dados) |
| Adicionar indice | Baixo | Nao (pode ser lento em tabelas grandes) |
| Remover indice | Medio | Nao (mas pode degradar performance) |
| Adicionar CHECK constraint | Medio | Sim (dados existentes devem satisfazer) |
| Mudar onDelete behavior | **Alto** | Nao (mas muda semantica de delete) |

---

## 2. Preparar a mudanca

### 2.1 Atualizar schema

- [ ] Editar `prisma/schema.prisma`
- [ ] Manter convencoes:
  - PK: `String @id @default(uuid())`
  - Timestamps: `createdAt DateTime @default(now()) @map("created_at")`
  - FK naming: `userId String @map("user_id")`
  - Tabela: `@@map("snake_case_plural")`
  - Monetario: `Decimal @db.Decimal(15, 2)`
  - Relacoes: `onDelete: Cascade` (exceto tabelas efemeras)
- [ ] Adicionar `@@index` em toda FK nova
- [ ] Se model novo tem campo Decimal, adicionar conversao no `$extends` em `lib/prisma.ts`

### 2.2 Gerar migracao

```bash
# Desenvolvimento (interativo — precisa de TTY)
npx prisma migrate dev --name <nome_descritivo>

# Producao (nao-interativo)
npx prisma migrate deploy
```

- [ ] Revisar o SQL gerado em `prisma/migrations/<timestamp>_<nome>/migration.sql`
- [ ] Se precisar de SQL manual (CHECK constraint, RENAME VALUE), editar o `migration.sql` ANTES de aplicar

### 2.3 SQL manual comum

**Adicionar CHECK constraint:**
```sql
ALTER TABLE tabela ADD CONSTRAINT chk_nome CHECK (expressao);
```

**Renomear enum value (sem perda de dados):**
```sql
ALTER TYPE "NomeEnum" RENAME VALUE 'antigo' TO 'novo';
```

**Adicionar enum value:**
```sql
ALTER TYPE "NomeEnum" ADD VALUE 'novo';
```

**Popular campo NOT NULL antes de tornar obrigatorio:**
```sql
-- 1. Adicionar como nullable
ALTER TABLE tabela ADD COLUMN campo tipo;
-- 2. Popular dados
UPDATE tabela SET campo = valor_default WHERE campo IS NULL;
-- 3. Tornar NOT NULL
ALTER TABLE tabela ALTER COLUMN campo SET NOT NULL;
```

---

## 3. Atualizar codigo

Seguir a lista de dependentes do passo 1.1, na ordem:

### 3.1 Camada de dados
- [ ] `lib/prisma.ts` — adicionar conversao `$extends` se novo campo Decimal
- [ ] `types/api.ts` — atualizar interfaces de request/response

### 3.2 Camada de servicos
- [ ] `lib/api/validators.ts` — adicionar/atualizar validacao para novos campos
- [ ] `lib/services/` — atualizar servicos afetados

### 3.3 Camada de API
- [ ] `app/api/*/route.ts` — atualizar rotas afetadas (create, update, read)
- [ ] Validar input do campo novo em POST/PUT routes

### 3.4 Integracoes
- [ ] _Adaptar conforme o projeto (IA tools, bots, webhooks, etc.)_

### 3.5 Camada de client
- [ ] `lib/hooks/` — atualizar hooks de fetch e tipagem
- [ ] `components/` — atualizar formularios, listas, metricas

---

## 4. Verificar

### 4.1 Build
- [ ] `npx prisma generate` — client regenerado sem erros
- [ ] `npm run build` — build passa sem erros de tipo
- [ ] `npm run lint` — sem erros de lint

### 4.2 Integridade
- [ ] Dados existentes satisfazem novas constraints?
- [ ] Cascade deletes continuam corretos?

### 4.3 Documentacao
- [ ] Atualizar `docs/data/data_dictionary.md` com novos campos/models
- [ ] Regenerar DBML e ERD: `npx prisma generate`
- [ ] Se decisao arquitetural nova, criar ADR em `docs/architecture/adr/`
- [ ] Atualizar `CLAUDE.md` se afeta convencoes ou padroes

---

## 5. Deploy

### 5.1 Ordem de deploy

Para mudancas **nao-breaking** (novo campo nullable, novo model, novo indice):
1. Deploy do codigo + migracao juntos

Para mudancas **breaking** (remover campo, mudar tipo, renomear):
1. Deploy 1: adicionar campo novo (convive com antigo)
2. Migrar dados: popular campo novo a partir do antigo
3. Deploy 2: codigo passa a usar campo novo
4. Deploy 3: remover campo antigo

### 5.2 Rollback

- [ ] A migracao e reversivel? (Prisma nao gera down migrations automaticamente)
- [ ] Se nao reversivel, preparar SQL de rollback manual ANTES de aplicar
- [ ] Testar rollback em ambiente de desenvolvimento

---

## Exemplo pratico

**Cenario:** Adicionar campo `notes` (texto opcional) a um model.

```
1. Analise de impacto:
   - API routes: GET/POST/PUT — precisa incluir no create/update
   - Hooks: tipagem precisa do campo
   - Components: adicionar textarea no formulario
   - Types: atualizar interface

2. Schema:
   notes String? @db.Text

3. Migracao:
   npx prisma migrate dev --name add_notes
   → ALTER TABLE tabela ADD COLUMN notes TEXT;

4. Codigo (ordem):
   - types/api.ts: adicionar notes?: string
   - POST route: incluir notes no create
   - PUT route: incluir notes no update
   - GET route: ja vem automaticamente
   - Hook: tipagem atualizada automaticamente pelo Prisma
   - Componente: adicionar campo textarea
   - data_dictionary.md: adicionar linha

5. Verificar:
   - prisma generate ✓
   - npm run build ✓
   - npm run lint ✓
   - Dados existentes: notes=NULL, OK

6. Deploy: campo nullable, deploy unico.
```

---

## Referencia rapida: convencoes de schema

| Aspecto | Convencao |
|---------|-----------|
| PK | `String @id @default(uuid())` |
| FK | `nomeId String @map("nome_id")` + `@@index([nomeId])` |
| Tabela | `@@map("snake_case_plural")` |
| Coluna | `@map("snake_case")` |
| Timestamp | `createdAt DateTime @default(now()) @map("created_at")` |
| Updated | `updatedAt DateTime @updatedAt @map("updated_at")` |
| Monetario | `Decimal @db.Decimal(15, 2)` + conversao automatica |
| Texto longo | `String @db.Text` |
| Relacao | `onDelete: Cascade` (exceto tabelas efemeras) |
| Enum values | lowercase |
| Boolean | `@default(false)` |
