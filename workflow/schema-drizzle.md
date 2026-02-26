# Checklist: Mudança de Schema (Drizzle)

Processo para qualquer alteração em `database/schema.ts` — novo model, novo campo, renomear coluna, adicionar índice, etc.

> Se o projeto usa Prisma, ver `workflow/schema-prisma.md`.

---

## 1. Análise de impacto

Antes de tocar no schema, mapear o que será afetado.

### 1.1 Identificar dependentes

Para o model/campo que será alterado, verificar:

- [ ] **Repositories** (`app/repositories/`) — quais funções fazem queries com esse campo?
- [ ] **Loaders / Actions** — algum loader ou action usa esse campo diretamente?
- [ ] **Componentes / hooks** — algum componente renderiza ou filtra por esse campo?
- [ ] **Schemas Zod** (`app/lib/schemas/`) — existe validação para esse campo?
- [ ] **Types inferidos** — tipos derivados de `typeof table.$inferSelect` precisam de update?
- [ ] **Integrações** — adaptar conforme o projeto (webhooks, bots, APIs de terceiros)

### 1.2 Classificar o risco

| Tipo de mudança | Risco | Requer migração de dados? |
|-----------------|-------|--------------------------|
| Adicionar coluna nullable | Baixo | Não |
| Adicionar coluna NOT NULL com default | Baixo | Não |
| Adicionar coluna NOT NULL sem default | **Alto** | Sim (popular antes) |
| Adicionar nova tabela | Baixo | Não |
| Renomear coluna | **Alto** | Sim (sem rename nativo no SQLite) |
| Remover coluna | Médio | Sim (verificar dependentes) |
| Remover tabela | **Alto** | Sim (verificar cascades e FKs) |
| Mudar tipo de coluna | **Alto** | Sim (recriar tabela no SQLite) |
| Adicionar índice | Baixo | Não (pode ser lento em tabelas grandes) |
| Mudar onDelete behavior | **Alto** | Não (mas muda semântica) |

> **Atenção SQLite / D1:** SQLite não suporta `ALTER TABLE ... RENAME COLUMN` em versões antigas e não suporta `DROP COLUMN` em D1. Para renomear ou remover, é necessário recriar a tabela.

---

## 2. Preparar a mudança

### 2.1 Atualizar schema

Editar `database/schema.ts` seguindo as convenções:

```typescript
import { index, integer, sqliteTable, text } from "drizzle-orm/sqlite-core";
import { id, money, timestamps } from "./utils";

export const myTable = sqliteTable(
  "my_table",                          // nome da tabela em snake_case
  {
    ...id(),                           // id: text UUID v7, PK
    organizationId: text("organization_id")
      .notNull()
      .references(() => organizations.id, { onDelete: "cascade" }),

    name: text("name").notNull(),
    description: text("description"),  // nullable — omitir .notNull()
    status: text("status", { enum: ["active", "inactive"] })
      .notNull()
      .default("active"),

    ...money("price"),                 // priceAmount: integer + priceCurrency: text
    ...timestamps(),                   // createdAt + updatedAt em milliseconds
  },
  (t) => [
    index("idx_my_table_organization_id").on(t.organizationId),  // índice em toda FK
  ]
);
```

**Helpers disponíveis em `database/utils/`:**

| Helper | Gera | Quando usar |
|--------|------|-------------|
| `id()` | `id: text UUID v7, primaryKey` | Toda tabela |
| `timestamps()` | `createdAt + updatedAt` em milliseconds | Toda tabela |
| `money(field)` | `fieldAmount: integer` + `fieldCurrency: text` | Todo valor monetário |

**Regras:**
- [ ] Toda FK recebe `index()` na mesma tabela
- [ ] Toda tabela de negócio tem `organizationId` com `onDelete: "cascade"`
- [ ] Nomes de colunas em `snake_case` (primeiro arg do helper de coluna)
- [ ] Enums como `text({ enum: [...] })` — valores em lowercase
- [ ] Booleanos como `integer({ mode: "boolean" }).default(false)`
- [ ] Datas como `integer({ mode: "timestamp_ms" })` (milliseconds)

### 2.2 Gerar a migração

```bash
# Gerar arquivo SQL de migração (não aplica ainda)
npx drizzle-kit generate

# Revisar o SQL gerado em drizzle/<timestamp>_<nome>.sql
# SEMPRE revisar antes de aplicar
```

### 2.3 Revisar o SQL gerado

- [ ] Abrir `drizzle/<timestamp>_<nome>.sql`
- [ ] Verificar que as operações estão corretas
- [ ] Se precisar de SQL manual (popular dados antes de NOT NULL, recriar tabela), editar o arquivo ANTES de aplicar

**SQL manual comum:**

```sql
-- Popular coluna antes de tornar NOT NULL
UPDATE my_table SET column = 'default_value' WHERE column IS NULL;

-- Recriar tabela para renomear coluna (SQLite não tem RENAME COLUMN em D1)
CREATE TABLE my_table_new (...);
INSERT INTO my_table_new SELECT old_col AS new_col, ... FROM my_table;
DROP TABLE my_table;
ALTER TABLE my_table_new RENAME TO my_table;
```

### 2.4 Aplicar a migração

```bash
# D1 local (desenvolvimento)
npx wrangler d1 migrations apply DB --local

# D1 remoto (produção) — confirmar antes de rodar
npx wrangler d1 migrations apply DB --remote

# Postgres / Turso (via drizzle-kit)
npx drizzle-kit migrate
```

---

## 3. Atualizar código

Seguir lista de dependentes do passo 1.1, na ordem:

### 3.1 Camada de dados
- [ ] `database/schema.ts` — atualizado (passo 2.1)
- [ ] Tipos inferidos: Drizzle infere automaticamente via `typeof table.$inferSelect` — verificar se há tipos manuais que precisam de update

### 3.2 Repositories
- [ ] `app/repositories/*.repository.ts` — atualizar queries que usam o campo alterado
- [ ] Se campo novo: adicionar nas funções `create` e `update` do repository

### 3.3 Validação
- [ ] `app/lib/schemas/` — atualizar schemas Zod correspondentes

### 3.4 Integrações
- [ ] Adaptar conforme o projeto (webhooks, bots, etc.)

### 3.5 UI
- [ ] Atualizar formulários, listas e visualizações que exibem o campo

---

## 4. Verificar

- [ ] `npx drizzle-kit generate` — sem erros de schema
- [ ] Build passa sem erros de tipo (`bun run build` ou `npm run build`)
- [ ] Lint sem erros
- [ ] Dados existentes satisfazem novas constraints?
- [ ] Cascade deletes continuam corretos?

---

## 5. Documentar

- [ ] `docs/data/data_dictionary.md` — atualizar com novos campos/models
- [ ] `CLAUDE.md` do projeto — se afeta convenções ou padrões
- [ ] Se decisão arquitetural nova, criar ADR em `docs/architecture/adr/`

---

## 6. Deploy

### Mudanças não-breaking (campo nullable, nova tabela, novo índice)
1. Deploy do código + migração juntos

### Mudanças breaking (renomear, remover, mudar tipo)
1. **Deploy 1:** adicionar campo novo (convive com antigo)
2. **Migrar dados:** popular campo novo a partir do antigo
3. **Deploy 2:** código passa a usar campo novo
4. **Deploy 3:** remover campo antigo

> Para D1, sempre rodar `--local` antes de `--remote`. Não há down migrations automáticas — preparar SQL de rollback manual antes de aplicar em produção.

---

## Referência rápida: convenções de schema Drizzle

| Aspecto | Convenção |
|---------|-----------|
| PK | `...id()` (UUID v7 via helper) |
| FK | `references(() => table.id, { onDelete: "cascade" })` + `index()` |
| Nome da tabela | `snake_case_plural` (primeiro arg de `sqliteTable`) |
| Nome da coluna | `snake_case` (primeiro arg de cada helper de coluna) |
| Timestamps | `...timestamps()` (createdAt + updatedAt em ms) |
| Monetário | `...money("fieldName")` (amount integer + currency text) |
| Texto longo | `text("column")` (SQLite não tem TEXT vs VARCHAR) |
| Boolean | `integer({ mode: "boolean" }).default(false)` |
| Data | `integer({ mode: "timestamp_ms" })` |
| Enum | `text({ enum: ["a", "b"] })` — valores em lowercase |
| Índice em FK | Obrigatório — `index("idx_table_fk").on(t.fkField)` |
