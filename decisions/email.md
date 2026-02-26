# Decisão: Email Provider

Define o serviço para envio de e-mails transacionais e marketing.

## Opções

### Resend

**Escolha quando:**
- Stack é TypeScript — Resend tem SDK moderno e React Email para templates
- Projeto novo — melhor DX do mercado atualmente para devs
- Volume moderado (free tier: 3.000 e-mails/mês, 100/dia)

**Trade-offs:**
- Mais novo — histórico de deliverability menor que Postmark/SES
- Free tier com limite diário (100/dia) pode ser restritivo em staging

**Segue com:**
```typescript
import { Resend } from "resend";
import { InvoiceEmail } from "@/emails/invoice";

const resend = new Resend(env.RESEND_API_KEY);

await resend.emails.send({
  from: "Acme <noreply@acme.com>",
  to: client.email,
  subject: `Fatura #${invoice.number}`,
  react: <InvoiceEmail invoice={invoice} />,
});
```

```tsx
// emails/invoice.tsx (React Email)
import { Html, Body, Heading, Text } from "@react-email/components";

export function InvoiceEmail({ invoice }: { invoice: Invoice }) {
  return (
    <Html>
      <Body>
        <Heading>Fatura #{invoice.number}</Heading>
        <Text>Valor: {formatMoney(invoice.total)}</Text>
      </Body>
    </Html>
  );
}
```

---

### Postmark

**Escolha quando:**
- Deliverability é prioridade máxima — Postmark tem histórico sólido
- Transacionais críticos: reset de senha, confirmação de pagamento
- Volume alto — preço competitivo acima de 10k e-mails/mês

**Trade-offs:**
- Sem suporte a React Email nativamente — templates em HTML ou Handlebars
- SDK menos moderno que Resend
- Mais caro no free tier (100 e-mails/mês apenas)

---

### AWS SES

**Escolha quando:**
- Volume muito alto (centenas de milhares/mês) — SES é o mais barato ($0.10/1000)
- Infra já é AWS
- Aceita complexidade de setup: domínios verificados, DKIM, bounce handling, suppression list

**Trade-offs:**
- Setup complexo — DKIM, SPF, DMARC, suppression list manual
- Sem dashboard útil — logs no CloudWatch
- Deliverability depende da reputação do IP/domínio que você gerencia

---

### SendGrid

**Escolha quando:**
- Precisa de e-mail marketing (campanhas, listas, segmentação) junto com transacional
- Time de marketing usa a plataforma — tem editor visual

**Trade-offs:**
- API mais complexa que Resend
- Histórico de problemas de deliverability em planos menores
- Mais caro que SES e Resend para volume alto

---

## Resumo

| Critério | Resend | Postmark | AWS SES | SendGrid |
|----------|--------|----------|---------|----------|
| DX / SDK moderno | ✅ | ⚠️ | ❌ | ⚠️ |
| React Email | ✅ | ❌ | ❌ | ❌ |
| Deliverability | ⚠️ | ✅ | ⚠️ (autogerenciado) | ⚠️ |
| Custo (alto volume) | ⚠️ | ⚠️ | ✅ | ⚠️ |
| E-mail marketing | ❌ | ❌ | ❌ | ✅ |
| Free tier | ✅ | ⚠️ | ✅ | ✅ |

**Regra rápida:** Projeto novo TypeScript → Resend (melhor DX). Transacionais críticos com histórico → Postmark. Volume acima de 100k/mês → AWS SES. Precisa de marketing + transacional → SendGrid.
