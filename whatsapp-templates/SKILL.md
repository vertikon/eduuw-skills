---
name: whatsapp-templates
description: List, create, delete and AI-generate WhatsApp message templates in eduuw.
---

# WhatsApp Templates

## When to Use

Managing message templates — required to start conversations outside the 24h window. Base + auth: see `eduuw-rules`.

## Lifecycle

`DRAFT` → submit → Meta review → `APPROVED` (sendable) | `REJECTED`. Only **APPROVED** templates can be sent (`send-message` → template).

- **Category:** `MARKETING | UTILITY | AUTHENTICATION`.
- **Language:** e.g. `pt_BR`, `en`, `es`.
- **Variables:** use `{{1}}`, `{{2}}`… in the body; fill them at send time via `parameters`.

## List

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/templates \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# -> { "templates": [ { "name": "...", "language": "pt_BR", "status": "APPROVED", ... } ] }
```

## Create (submits for approval)

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/templates \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{
    "name": "confirmacao_matricula",
    "language": "pt_BR",
    "category": "UTILITY",
    "body": "Olá {{1}}, sua matrícula em {{2}} foi confirmada!",
    "footerText": "Equipe eduuw"
  }'
```
`name` must be lowercase snake_case (`[a-z0-9_]`). Returns the template in `PENDING`.

## Generate with AI (suggestion, does not create)

Describe the template in natural language; the API returns a suggested `name`/`category`/`body`/`footer` (requires the server's GLM to be configured). Review, then create with the call above.

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/templates/generate \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"prompt":"confirmar matrícula do aluno com nome e curso","category":"UTILITY","language":"pt_BR"}'
# -> { "name": "confirmacao_matricula", "category": "UTILITY", "language": "pt_BR", "body": "...", "footer": "..." }
```
503 → IA indisponível (GLM not configured on the server). Scope: `whatsapp:write`.

## Get / Delete

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/templates/<NAME> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/templates/<NAME> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Tips

- Keep MARKETING templates compliant (opt-out friendly) — Meta rejects spammy copy.
- For dynamic, per-recipient variables in bulk, use `broadcast-campaign`.
