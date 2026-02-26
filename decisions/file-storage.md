# Decisão: File Storage

Define onde arquivos enviados pelos usuários são armazenados. Impacta custo, latência e DX de upload.

## Opções

### Cloudflare R2

**Escolha quando:**
- Runtime é Cloudflare Workers + Pages
- Quer zero custo de egress (leitura de arquivos não cobra por GB transferido)
- Arquivos servidos via CDN do Cloudflare automaticamente

**Trade-offs:**
- Acoplado ao Cloudflare — migrar tem custo
- API S3-compatible mas com alguns comportamentos levemente diferentes
- Upload direto do browser exige Presigned URL ou Workers intermediário

**Segue com:**
```typescript
// workers/app.ts — binding configurado no wrangler.jsonc
const bucket = context.cloudflare.env.R2_BUCKET;

// Upload via Presigned URL (browser → R2 direto)
const url = await bucket.createMultipartUpload(key);

// Upload server-side
await bucket.put(key, fileBody, { httpMetadata: { contentType } });

// URL pública (configurar domínio público no dashboard do R2)
const publicUrl = `https://cdn.seudominio.com/${key}`;
```

---

### AWS S3

**Escolha quando:**
- Infra já é AWS ou requer máxima compatibilidade de ecossistema
- Precisa de features avançadas: lifecycle policies, versioning, replication, S3 Select
- Clientes ou compliance exigem AWS

**Trade-offs:**
- Custo de egress — cada GB lido cobra (~$0.09/GB)
- Setup de IAM + políticas de bucket tem curva
- Mais complexo que R2 para casos simples

**Segue com:**
- `@aws-sdk/client-s3` + `@aws-sdk/s3-request-presigner`
- Presigned URLs para upload direto do browser
- CloudFront na frente do S3 para CDN e reduzir custo de egress

---

### Uploadthing

**Escolha quando:**
- Stack é Next.js ou React Router + Node.js
- Quer a menor fricção possível: componente de upload + backend em poucas linhas
- MVP — sem precisar configurar bucket, policies e CDN manualmente

**Trade-offs:**
- Vendor lock-in — arquivos ficam na infra do Uploadthing
- Custo por GB armazenado e transferido (free tier: 2GB)
- Menos controle sobre storage, lifecycle e CDN

**Segue com:**
```typescript
// server
import { createUploadthing } from "uploadthing/next";
const f = createUploadthing();

export const uploadRouter = {
  imageUploader: f({ image: { maxFileSize: "4MB" } })
    .middleware(async ({ req }) => {
      const user = await getUser(req);
      return { userId: user.id };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      await saveFileUrl(metadata.userId, file.url);
    }),
};
```

---

### Supabase Storage

**Escolha quando:**
- Banco já é Supabase
- Quer RLS integrado com o storage (acesso baseado em auth)
- Precisa de imagens transformadas (resize, crop) via CDN

**Trade-offs:**
- Acoplado ao Supabase — migrar storage junto com banco
- Custo adicional acima do free tier

---

## Resumo

| Critério | R2 | S3 | Uploadthing | Supabase Storage |
|----------|----|----|-------------|-----------------|
| Egress gratuito | ✅ | ❌ | ⚠️ | ⚠️ |
| Cloudflare nativo | ✅ | ❌ | ❌ | ❌ |
| DX / setup | ⚠️ | ⚠️ | ✅ | ✅ |
| Ecossistema / integrações | ⚠️ | ✅ | ⚠️ | ⚠️ |
| Self-hosted | ❌ | ✅ (MinIO) | ❌ | ❌ |
| Transformação de imagem | ⚠️ | ⚠️ | ❌ | ✅ |

**Regra rápida:** Cloudflare Workers → R2. AWS / compliance → S3 + CloudFront. MVP rápido com Next.js → Uploadthing. Já usa Supabase → Supabase Storage.
