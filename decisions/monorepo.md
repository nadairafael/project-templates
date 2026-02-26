# Decisão: Monorepo

Define se o projeto usa repositório único para múltiplos pacotes/apps. Impacta DX de refactoring, CI e deploy.

> **Regra:** começar sem monorepo. Adicionar quando você tiver 2+ pacotes com código compartilhado real que já está sendo duplicado.

---

## Quando monorepo faz sentido

- App web + app mobile (React Native) compartilhando tipos e lógica de negócio
- Frontend + backend como pacotes separados com tipos compartilhados
- Design system como pacote isolado consumido por múltiplas apps
- CLI ou SDK público derivado do produto

---

## Opções

### Sem monorepo (padrão)

**Escolha quando:**
- Projeto é uma única app
- Equipe pequena (1-3 devs)
- MVP — adicionar estrutura de monorepo tem custo de setup e manutenção

**Segue com:**
- Repositório único, sem workspaces
- Tipos compartilhados em `app/lib/types/` dentro do mesmo projeto

---

### Turborepo

**Escolha quando:**
- 2+ apps ou pacotes no mesmo repo
- Deploy no Vercel — integração nativa com Remote Cache (builds incrementais no CI)
- Quer pipeline de tasks com dependências (build package antes do app)

**Trade-offs:**
- Configuração inicial tem curva
- Remote Cache requer conta Vercel ou Turborepo Cloud para funcionar em CI externo
- Overhead de gerenciar `package.json` por workspace

**Segue com:**
```
/
├── apps/
│   ├── web/          # Next.js ou React Router
│   └── mobile/       # Expo / React Native
├── packages/
│   ├── ui/           # Design system compartilhado
│   ├── types/        # Tipos compartilhados
│   └── config/       # tsconfig, eslint, etc.
├── turbo.json
└── package.json      # workspaces: ["apps/*", "packages/*"]
```

```json
// turbo.json
{
  "tasks": {
    "build": { "dependsOn": ["^build"], "outputs": [".next/**", "dist/**"] },
    "dev": { "persistent": true },
    "lint": {}
  }
}
```

---

### Nx

**Escolha quando:**
- Equipe grande (5+ devs) com múltiplos times
- Precisa de affected commands (rodar testes só nos pacotes afetados por um PR)
- Projeto enterprise com muitos apps e libs

**Trade-offs:**
- Mais poderoso que Turborepo mas muito mais complexo de configurar e manter
- Overkill para projetos pequenos e médios
- Curva de aprendizado alta

---

### pnpm workspaces (sem Turborepo)

**Escolha quando:**
- Quer workspaces simples sem orquestrador de tasks
- Monorepo pequeno — 2-3 pacotes sem pipelines complexas

**Trade-offs:**
- Sem cache de tasks — cada build recomeça do zero
- Sem grafo de dependências entre tasks

---

## Resumo

| Critério | Sem monorepo | Turborepo | Nx | pnpm workspaces |
|----------|-------------|-----------|-----|-----------------|
| Simplicidade de setup | ✅ | ⚠️ | ❌ | ✅ |
| Cache de builds | — | ✅ | ✅ | ❌ |
| Affected commands | — | ⚠️ | ✅ | ❌ |
| Integração Vercel | — | ✅ | ⚠️ | ⚠️ |
| Escala para equipes grandes | ❌ | ✅ | ✅ | ⚠️ |

**Regra rápida:** app única → sem monorepo. 2-3 pacotes com Vercel → Turborepo. Equipe grande / enterprise → Nx. Workspaces simples sem cache → pnpm workspaces.
