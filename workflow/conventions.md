# Convencoes do Projeto

Regras codificadas que todo código deve seguir. Preencher seções marcadas com _definir_ após as decisões de stack (ver `decisions/`).

---

## Idioma

| Contexto | Idioma | Exemplo |
|----------|--------|---------|
| Textos de UI (labels, mensagens, placeholders) | _definir_ (ex: pt-BR, en — ou ambos via i18n) | — |
| Mensagens de erro da API | _definir_ (ex: inglês para APIs públicas; idioma do usuário para apps consumer) | — |
| Nomes de variável, função, classe | Inglês | `getAccounts()`, `userId` |
| Commits | Inglês | `feat(api): add batch endpoint` |
| Documentação técnica | _definir_ (ex: inglês para open source; português para equipe local) | — |
| Comentários no código | Inglês (preferencial) | — |

---

## Estrutura de diretórios

_Preencher após scaffold. Adaptar conforme o framework escolhido._

```
_definir_
```

### Regras de organização (agnósticas de framework)

- **Um arquivo por hook** — `hooks/use-expenses.ts`, não `hooks/index.ts` com tudo junto
- **Um arquivo por serviço** — `services/balance.ts`, não `services/index.ts`
- **Componentes nomeados pelo recurso** — `add-expense-dialog.tsx`, não `dialog-1.tsx`
- **Não criar `index.ts` barrel exports** — importar diretamente do arquivo
- **Não criar utilitários genéricos prematuramente** — 3 linhas duplicadas é melhor que uma abstração prematura

---

## Naming

### Arquivos

| Tipo | Convenção | Exemplo |
|------|-----------|---------|
| Componente React | kebab-case | `add-expense-dialog.tsx` |
| Hook | `use-` prefix, kebab-case | `use-expenses.ts` |
| Serviço / utilitário | kebab-case | `expense-batch.ts`, `dates.ts` |
| Repository | kebab-case com sufixo | `clients.repository.ts` |
| Schema Zod | kebab-case | `client-schema.ts` |

### Código

| Tipo | Convenção | Exemplo |
|------|-----------|---------|
| Componente React | PascalCase | `AddExpenseDialog` |
| Hook | camelCase com `use` prefix | `useExpenses` |
| Função | camelCase | `getAccounts`, `applyBalanceChange` |
| Constante | UPPER_SNAKE_CASE | `RATE_LIMIT_CHAT`, `MAX_FILE_SIZE` |
| Variável | camelCase | `cardAccounts`, `nextCursor` |
| Tipo / Interface | PascalCase | `ExpenseInput`, `ApiResponse` |
| Enum values | lowercase | `{ active, inactive, pending }` |

---

## Design system

> Decisão: ver `decisions/ui.md`

### Design tokens

_Preencher com tokens extraídos das referências visuais coletadas na Fase 2 (ver seção "Referências visuais" em `decisions/ui.md`)._

| Token | Valor |
|-------|-------|
| Cor primária | _definir_ |
| Cor de fundo | _definir_ |
| Tipografia (font-family) | _definir_ |
| Raio de borda base | _definir_ |
| Espaçamento base | _definir_ |

_Se não houve referências visuais: usar defaults da lib escolhida e remover esta tabela._

### Se Tailwind + shadcn/ui

- Primitivos em `components/ui/` — não editar diretamente; reinstalar via `npx shadcn@latest add`
- Tokens gerados pelo shadcn: `--primary`, `--background`, `--muted`, `--border`, etc.
- Nunca usar valores hex, rgb ou classes arbitrárias onde existe token
- Ícones: `lucide-react`

### Se Tailwind + componentes próprios

- CSS variables para todos os tokens: `var(--color-*)`, `var(--space-*)`, `var(--radius-*)`
- Nunca hardcodar hex, rgb ou px onde existe variável
- Documentar cada componente reutilizável na tabela abaixo

### Componentes reutilizáveis

_Preencher conforme o projeto cria componentes._

| Componente | Arquivo | Quando usar |
|------------|---------|-------------|
| _adicionar conforme surgem_ | | |

### Classes CSS globais

_Documentar conforme surgem._

| Classe | Uso |
|--------|-----|
| _adicionar conforme surgem_ | |

---

## Componentes

### Regras universais

| Regra | Detalhe |
|-------|---------|
| Exports | Sempre named export — nunca `export default` |
| Props | Estender `ComponentProps<"element">` quando wrappa elemento HTML nativo |
| Icon buttons | Sempre `aria-label` descritivo — nunca ícone sem texto alternativo |
| Estado | Preferir `data-[state]:` sobre classes condicionais no JSX |

### Se Tailwind + shadcn/ui

- Variantes via `cva` (já presente no shadcn)
- Estado interativo via `data-[state]:`, `data-[disabled]:`, `data-[open]:` — padrão do Radix UI / Base UI
- Não sobrescrever componentes de `components/ui/` — compor ou criar novo arquivo em `components/`
- shadcn suporta tanto Radix UI quanto Base UI como camada headless — preferir Base UI para projetos com componentes de formulário complexos (Combobox, multi-select)

```typescript
import { Button, type ButtonProps } from "@/components/ui/button"

interface SaveButtonProps extends ButtonProps {
  saving?: boolean
}

export function SaveButton({ saving, ...props }: SaveButtonProps) {
  return <Button data-saving={saving || undefined} {...props} />
}
```

### Se Tailwind + design system próprio

- Camada headless: **Base UI** (preferência) ou Radix UI — nunca implementar dialog, select, combobox do zero
- Variantes via `tailwind-variants` — preferência sobre `cva` para projetos Tailwind (composição de slots, variantes responsivas)
- Estado via `data-[state]:` — evitar lógica de classe no JSX
- Acessibilidade vem da camada headless — não reimplementar `role`, `aria-*`, focus trap manualmente

```typescript
import { tv } from "tailwind-variants"
import * as BaseButton from "@base-ui-components/react/button"

const button = tv({
  base: "inline-flex items-center gap-2 rounded font-medium focus-visible:outline-offset-2",
  variants: {
    variant: {
      primary: "bg-primary text-primary-foreground hover:bg-primary/90",
      ghost: "hover:bg-muted",
    },
    size: {
      sm: "h-8 px-3 text-sm",
      md: "h-10 px-4",
    },
  },
  defaultVariants: { variant: "primary", size: "md" },
})

interface ButtonProps extends React.ComponentProps<typeof BaseButton.Root> {
  variant?: "primary" | "ghost"
  size?: "sm" | "md"
}

export function Button({ variant, size, className, ...props }: ButtonProps) {
  return <BaseButton.Root className={button({ variant, size, className })} {...props} />
}
```

---

## Formulários

> Decisão de biblioteca de forms: _definir_ (ex: `@conform-to/react` para SSR + progressive enhancement, `React Hook Form` para client-side — ver opções abaixo)

### @conform-to/react + Zod

Usar quando: SSR com progressive enhancement, React Router / Remix actions.

```typescript
// Schema i18n-aware
export const schema = (t: TFunction) =>
  z.object({
    name: z.string().min(1, t("validation.required")),
  });

// No componente
const [form, fields] = useForm({
  onValidate: ({ formData }) => parseWithZod(formData, { schema: schema(t) }),
});
```

### React Hook Form + Zod

Usar quando: client-side heavy, sem SSR form actions.

```typescript
const form = useForm<Schema>({
  resolver: zodResolver(schema),
});
```

---

## Dados client-side

> Decisão: _definir_ (ex: TanStack Query v5 para cache compartilhado e invalidação coordenada; loaders nativos do framework para apps simples sem cache client-side; SWR como meio-termo leve)

### TanStack Query

Usar quando: cache compartilhado entre componentes, invalidação coordenada, refetch automático.

```typescript
// Query keys em arquivo centralizado (query-client.ts)
export const keys = {
  clients: {
    all: () => ["clients"] as const,
    byId: (id: string) => ["clients", id] as const,
  },
};

const { data } = useQuery({
  queryKey: keys.clients.all(),
  queryFn: () => fetch("/api/clients").then(r => r.json()),
});
```

### Fetch hook com AbortController

Usar quando: sem biblioteca de cache, componente isolado.

```typescript
export function useRecurso() {
  const [data, setData] = useState<Recurso[]>([]);
  const [loading, setLoading] = useState(true);

  const fetchData = useCallback(async (signal: AbortSignal) => {
    try {
      const res = await fetch("/api/recurso", { signal });
      if (!res.ok) throw new Error();
      setData(await res.json());
    } catch (err) {
      if (err instanceof Error && err.name === "AbortError") return;
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    const controller = new AbortController();
    fetchData(controller.signal);
    return () => controller.abort();
  }, [fetchData]);

  return { data, loading, refetch: fetchData };
}
```

**Regras (independente da abordagem):**
- AbortController em todo fetch com cleanup no unmount
- Silenciar `AbortError` — não é um erro real
- Optimistic updates: atualizar state local imediatamente, rollback on error

---

## API / Actions

### Ordem de execução em qualquer handler

```
1. Autenticação — rejeitar antes de qualquer outra coisa
2. Autorização — verificar permissão no recurso específico
3. Rate limiting — com constante centralizada
4. Parse do input — try/catch em JSON, validar com Zod
5. Lógica de negócio
6. Resposta
```

### Regras

| Regra | Detalhe |
|-------|---------|
| Auth | Verificar autenticação no início de toda rota/action protegida |
| Tenant scope | Toda query inclui `organizationId` / `userId` — nunca buscar sem escopo |
| Rate limit | Constante centralizada em `config.ts`, nunca número mágico inline |
| Erros ao cliente | Mensagens genéricas, sem detalhes internos (stack trace, SQL, paths) |
| Erros internos | Logger estruturado, stack trace só em dev |
| Transações | Para operações multi-step — sempre com timeout |
| JSON parse | `try/catch` → retornar erro 400 |

---

## Database

> Decisão de ORM: ver `decisions/orm.md`

### Se Drizzle

- Schema em `database/schema.ts`
- Migrations em `drizzle/`
- Helpers obrigatórios: `id()`, `timestamps()`, `money(field)`
- Toda FK recebe `index()` na mesma tabela
- Workflow de mudança: `workflow/schema-drizzle.md`

```typescript
// Exemplo de tabela
export const clients = sqliteTable("clients", {
  ...id(),
  organizationId: text("organization_id")
    .notNull()
    .references(() => organizations.id, { onDelete: "cascade" }),
  name: text("name").notNull(),
  ...timestamps(),
}, (t) => [
  index("idx_clients_organization_id").on(t.organizationId),
]);
```

### Se Prisma

- Schema em `prisma/schema.prisma`
- Migrations em `prisma/migrations/`
- Workflow de mudança: `workflow/schema-prisma.md`

```prisma
model Client {
  id             String   @id @default(uuid())
  organizationId String   @map("organization_id")
  name           String
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([organizationId])
  @@map("clients")
}
```

---

## Acessibilidade

| Regra | Implementação |
|-------|---------------|
| Skip link | `<a href="#main-content">` no layout, `<main id="main-content">` em cada página |
| Dialog | `role="dialog"`, `aria-modal="true"`, `aria-labelledby`, focus trap, Escape fecha |
| Focus visible | `focus-visible` outline com offset |
| Disabled | `opacity`, `cursor: not-allowed`, `pointer-events: none` |
| Motion | `prefers-reduced-motion: reduce` desabilita animações |

---

## Responsividade

| Regra | Detalhe |
|-------|---------|
| Approach | Mobile-first — estilizar para mobile, override no breakpoint desktop |
| Breakpoint | _definir_ único ponto de quebra (ex: `768px` mobile/tablet, `1024px` tablet/desktop) |
| Dialog | `max-width: 100%` para não estourar em mobile |
| Testar | Viewports: 375px (mobile) e 1280px (desktop) |

---

## Commits

Conventional Commits. Mensagem em inglês, modo imperativo, max 72 chars.

```
<type>(scope): <description>
```

### Types

| Type | Quando usar |
|------|-------------|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `refactor` | Mudança sem alterar comportamento |
| `style` | Formatação, CSS, sem mudança de lógica |
| `docs` | Documentação |
| `chore` | Tarefas de manutenção, configs |
| `perf` | Otimização de performance |
| `build` | Build, deps, CI |

### Scopes

_Definir conforme a estrutura do projeto._

| Scope | Abrange |
|-------|---------|
| `auth` | Login, sessão |
| `api` | Handlers, actions, services |
| `ui` | Componentes, design system |
| `db` | Schema, migrations |
| `deps` | Dependências |
| _definir_ | — |

---

## Erros comuns

| Erro | Como evitar |
|------|-------------|
| Query sem escopo de tenant | Sempre incluir `organizationId` / `userId` na cláusula WHERE |
| Fetch sem AbortController | Sempre incluir cleanup no unmount |
| Transação sem timeout | Sempre incluir timeout explícito |
| JSON parse sem try/catch | Sempre envolver em try/catch |
| FK sem índice | Toda FK nova recebe índice |
| `console.error` em handler | Usar logger estruturado |
| Editar arquivo sem ler antes | Sempre ler o arquivo antes de editar |
| Abstração prematura | Não abstrair antes de ter 3 instâncias concretas |
