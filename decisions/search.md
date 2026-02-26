# Decisão: Search

Define como busca textual é implementada. Relevante quando busca é feature core do produto.

> Antes de adicionar um serviço de busca externo, avaliar se **Postgres full-text search** ou **SQLite FTS5** resolve. Para a maioria dos SaaS com até 100k registros, é suficiente.

---

## Opções

### Postgres Full-Text Search

**Escolha quando:**
- Banco é Postgres e o volume de dados é moderado (até ~500k registros)
- Busca é feature secundária — não é o diferencial do produto
- Quer zero infra extra

**Trade-offs:**
- Sem typo tolerance nativa — "prodto" não acha "produto"
- Ranking menos sofisticado que Algolia/Typesense
- Performance degrada em tabelas muito grandes sem tuning de índices

**Segue com:**
```sql
-- Índice GIN para full-text
CREATE INDEX idx_products_fts ON products
  USING GIN (to_tsvector('portuguese', name || ' ' || description));

-- Query
SELECT * FROM products
WHERE to_tsvector('portuguese', name || ' ' || description) @@ plainto_tsquery('portuguese', $1)
ORDER BY ts_rank(...) DESC;
```

```typescript
// Drizzle
const results = await db.select().from(products)
  .where(sql`to_tsvector('portuguese', ${products.name}) @@ plainto_tsquery('portuguese', ${query})`);
```

---

### SQLite FTS5 (D1 / Turso)

**Escolha quando:**
- Banco é D1 ou Turso (SQLite)
- Busca simples sem typo tolerance

**Segue com:**
```sql
CREATE VIRTUAL TABLE products_fts USING fts5(name, description, content=products);

-- Query
SELECT products.* FROM products
JOIN products_fts ON products.id = products_fts.rowid
WHERE products_fts MATCH 'query*';
```

---

### Meilisearch

**Escolha quando:**
- Busca é feature central com typo tolerance, facets e ranking customizado
- Dados têm caracteres especiais ou idiomas com acentuação (pt-BR, fr, de)
- Prefere self-host (Meilisearch Cloud disponível se não)

**Trade-offs:**
- Infra extra — instância Meilisearch separada (Railway, Fly.io, ou cloud)
- Sincronização: escrever no banco e indexar no Meilisearch (dupla escrita ou via webhook)
- Self-host: RAM proporcional ao volume de dados indexados

**Segue com:**
```typescript
import { MeiliSearch } from "meilisearch";
const client = new MeiliSearch({ host: env.MEILISEARCH_URL, apiKey: env.MEILISEARCH_KEY });

// Indexar ao criar/atualizar
await client.index("products").addDocuments([{ id, name, description }]);

// Buscar
const results = await client.index("products").search(query, {
  attributesToHighlight: ["name"],
  limit: 20,
});
```

---

### Typesense

**Escolha quando:**
- Meilisearch mas com Typesense Cloud (managed, sem self-host)
- Quer API mais próxima do Algolia com custo menor

**Trade-offs:**
- Typesense Cloud: custo por nó (~$0.02/hora para o menor)
- Self-host disponível mas menos documentado que Meilisearch

---

### Algolia

**Escolha quando:**
- Busca é o produto principal (marketplace, e-commerce com catálogo grande)
- Quer o melhor DX de integração com React (InstantSearch)
- Budget existe — Algolia é significativamente mais caro

**Trade-offs:**
- Caro: free tier limitado (10k registros, 10k buscas/mês); planos pagos a partir de $50+/mês
- Vendor lock-in mais profundo que Meilisearch/Typesense

---

## Resumo

| Critério | PG FTS | SQLite FTS5 | Meilisearch | Typesense | Algolia |
|----------|--------|-------------|-------------|-----------|---------|
| Typo tolerance | ❌ | ❌ | ✅ | ✅ | ✅ |
| Zero infra extra | ✅ | ✅ | ❌ | ❌ | ❌ |
| Acentuação pt-BR | ⚠️ | ⚠️ | ✅ | ✅ | ✅ |
| Self-host | ✅ | ✅ | ✅ | ✅ | ❌ |
| Custo | ✅ | ✅ | ⚠️ | ⚠️ | ❌ |
| DX / React UI | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ✅ |

**Regra rápida:** busca secundária em Postgres → FTS nativo. Busca secundária em D1 → SQLite FTS5. Busca central com typo tolerance → Meilisearch (self-host) ou Typesense Cloud. E-commerce / marketplace → Algolia.
