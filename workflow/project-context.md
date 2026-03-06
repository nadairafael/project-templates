# Contexto do Projeto

Documento para dar contexto completo ao agente sobre o que esta sendo construido. Diferente do overview.md (resumo de arquitetura), este foco e no **produto**: o que o usuario ve, como navega, qual o nivel de acabamento esperado.

_Preencher na Fase 1 do bootstrap, antes das decisoes de stack._

---

## O que e o produto

_Descrever em 3-5 frases como se estivesse explicando para alguem que nunca ouviu falar do projeto. Incluir: quem usa, qual problema resolve, como resolve._

```
_definir_
```

---

## Personas

_Quem vai usar o sistema. Para cada persona: nome, contexto, o que espera._

| Persona | Contexto | O que espera |
|---------|----------|--------------|
| _definir_ | _definir_ | _definir_ |

---

## Fluxos principais

_Descrever os fluxos que o usuario percorre no MVP. Cada fluxo e uma sequencia de acoes._

### Fluxo 1: _definir nome_

```
1. Usuario acessa _definir_
2. _definir_
3. _definir_
```

### Fluxo 2: _definir nome_

```
1. _definir_
2. _definir_
```

_Adicionar quantos fluxos forem necessarios._

---

## Mapa de rotas / telas

_Listar as rotas da aplicacao com o que cada uma faz. Isso define o shell inicial do projeto._

| Rota | Tela | O que faz | Auth? |
|------|------|-----------|-------|
| `/` | Home / Landing | _definir_ | _definir_ |
| `/login` | Login | _definir_ | Nao |
| `/dashboard` | Dashboard | _definir_ | Sim |
| _definir_ | _definir_ | _definir_ | _definir_ |

_Adicionar todas as rotas previstas para o MVP._

---

## Modelos de dados simplificado

_Descrever as entidades principais e como se relacionam — sem se preocupar com tipos ou campos exatos (isso vai pro data_dictionary.md). Aqui e o modelo mental._

```
_definir_ — ex:
- Usuario tem muitas Transacoes
- Transacao pertence a uma Categoria
- Categoria pertence a um Usuario
```

---

## Tom e personalidade

_Como o produto se comunica com o usuario. Isso impacta textos de UI, mensagens de erro, empty states._

| Aspecto | Definicao |
|---------|-----------|
| Tom geral | _definir_ (ex: profissional e direto, casual e amigavel, tecnico e preciso) |
| Mensagens de erro | _definir_ (ex: "Algo deu errado" vs "Nao foi possivel salvar: conexao perdida") |
| Empty states | _definir_ (ex: ilustracao + CTA, ou texto simples) |
| Idioma da UI | _definir_ |

---

## Nivel de refino visual

_Define o que se espera do design neste momento. Isso evita que o agente gere algo muito cru ou muito polido para a fase atual._

| Aspecto | Expectativa |
|---------|-------------|
| Fidelidade geral | _definir_ (ex: wireframe funcional, prototipo com estilo, producao final) |
| Responsividade | _definir_ (ex: mobile-first, desktop-only, ambos) |
| Animacoes | _definir_ (ex: nenhuma, transicoes basicas, micro-interacoes) |
| Dark mode | _definir_ (ex: nao, sim desde o inicio, depois do MVP) |
| Densidade de informacao | _definir_ (ex: espacado e limpo, denso tipo dashboard financeiro) |

_Se tiver referencias visuais (Figma, imagens, URLs), registrar em `decisions/ui.md` na secao "Referencias visuais"._

---

## Restricoes e premissas

_Coisas que o agente precisa saber para nao tomar decisoes erradas._

- _definir_ — ex: "O backend ja existe em outra linguagem, estamos so fazendo o frontend"
- _definir_ — ex: "Precisa funcionar offline"
- _definir_ — ex: "Nao pode usar servicos pagos — apenas free tier"
- _definir_ — ex: "Dados sensiveis — precisa de criptografia em repouso"

---

## Fora de escopo (MVP)

_O que NAO vai ser construido agora. Diferente dos non-goals do overview.md (que sao permanentes), aqui sao coisas que podem vir depois._

- _definir_ — ex: "Notificacoes push — vem no v2"
- _definir_ — ex: "Relatorios exportaveis — vem no v2"
- _definir_ — ex: "Multi-idioma — vem no v2"
