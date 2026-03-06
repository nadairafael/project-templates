# Decisão: UI e Componentes

Define como a interface é construída. Impacta velocidade de desenvolvimento e flexibilidade visual.

## Opções

### Tailwind v4 + shadcn/ui

**Escolha quando:**
- Velocidade de desenvolvimento é prioridade
- Design segue padrões comuns de SaaS (dashboards, forms, tabelas, modais)
- Não há design system proprietário com tokens rígidos
- Equipe já conhece Tailwind

**Trade-offs:**
- Visual "SaaS genérico" — diferenciação de marca exige mais esforço de customização
- Componentes são copiados para o repo (você mantém o código, não é uma lib)
- Tailwind v4 ainda jovem — alguns plugins e ferramentas ainda em adaptação
- shadcn gera componentes em `components/ui/` que não devem ser editados diretamente

**Segue com:**
- `components/ui/` — primitivos shadcn (não editar; re-instalar via CLI se precisar atualizar)
- `components/` — compostos e componentes específicos de página
- Adicionar componente: `npx shadcn@latest add <component>`
- Tokens: CSS variables geradas pelo shadcn (`--primary`, `--background`, `--muted`, etc.)
- Ícones: `lucide-react` (padrão shadcn)
- Acessibilidade: Radix UI por baixo dos componentes shadcn (focus trap, aria, etc.)

---

### Tailwind v4 + componentes próprios

**Escolha quando:**
- Design system proprietário com brand forte (tokens específicos, identidade visual única)
- Radix UI como primitivo de acessibilidade, mas UI completamente customizada
- Equipe tem capacidade de manter a biblioteca de componentes

**Trade-offs:**
- Mais tempo de desenvolvimento inicial
- Acessibilidade é responsabilidade da equipe — usar Radix UI como base obrigatória
- Cada componente novo precisa ser documentado em `conventions.md`

**Segue com:**
- `components/ui/` — primitivos próprios construídos sobre Radix UI
- CSS variables para tokens de design (`--color-brand-*`, `--space-*`, `--radius-*`)
- Documentar cada componente reutilizável em `conventions.md` (nome, arquivo, quando usar)

---

### CSS Modules + componentes próprios

**Escolha quando:**
- Tailwind não é opção (decisão de equipe ou cliente)
- App tem design muito específico (editorial, marketing pesado, fora do padrão SaaS)
- Equipe prefere CSS puro e tem experiência com design systems

**Trade-offs:**
- Mais CSS para escrever e manter
- Sem utilitários — layout, spacing e tipografia tudo manual
- Produtividade menor em telas novas

**Segue com:**
- CSS variables para tokens no `:root`
- Mobile-first, breakpoint único definido em `conventions.md`
- Radix UI ainda recomendado para primitivos acessíveis (Dialog, Select, etc.)

---

## Ícones

| Biblioteca | Quando usar |
|-----------|-------------|
| `lucide-react` | Padrão ao usar shadcn — consistente com os componentes |
| `@phosphor-icons/react` | Mais variedade de estilos (thin, bold, fill, duotone) — bom para custom UI |
| `heroicons` | Se já usa Tailwind UI oficialmente |

---

## Resumo

| Critério | Tailwind + shadcn | Tailwind + custom | CSS Modules |
|----------|-------------------|-------------------|-------------|
| Velocidade de desenvolvimento | ✅ | ⚠️ | ❌ |
| Flexibilidade visual | ⚠️ | ✅ | ✅ |
| Acessibilidade out-of-the-box | ✅ | ✅ (com Radix) | ⚠️ |
| Manutenção a longo prazo | ✅ | ⚠️ | ❌ |
| Diferenciação de marca | ⚠️ | ✅ | ✅ |

**Regra rápida:** SaaS / dashboard / internal tool → Tailwind + shadcn. Brand forte com design system próprio → Tailwind + custom. Sem Tailwind por requisito → CSS Modules.

---

## Referências visuais

Após definir a abordagem de componentes, coletar referências de design antes de começar o scaffold. Isso evita UI genérica e orienta decisões de tokens (cores, tipografia, espaçamento).

**Pedir ao menos uma das opções abaixo:**

- **Figma** — link para o arquivo ou frame específico
- **Imagens / screenshots** — prints de telas, moodboards, ou exemplos de apps com visual parecido
- **URLs de referência** — sites ou produtos cujo design se quer aproximar
- **Descrição textual** — ex: "minimalista, fundo escuro, tipografia grande, sem bordas visíveis"

**Se não houver referências:** registrar como "sem referência visual definida — usar defaults da lib escolhida" e seguir.

**O que fazer com as referências:**
- Extrair os tokens principais: paleta de cores, tipografia, raio de borda, espaçamento base
- Registrar em `conventions.md` na seção de design tokens
- Se usar shadcn: documentar as customizações das CSS variables (`--primary`, `--radius`, etc.)
- Se usar custom: criar o arquivo de tokens antes de qualquer componente
