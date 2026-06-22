---
name: phone-numbers
description: Manage connected WhatsApp numbers — list, quality/health, set active, deregister/migrate.
---

# Phone Numbers

## When to Use

Inspect and manage the WhatsApp numbers connected to a tenant. Base + auth: see `eduuw-rules`.

## Model

A tenant can connect **several numbers**; exactly **one is active** (used by default for sends). Each number = `{ phoneNumberId, businessAccountId (WABA), kind, isActive, webhookConfigured }`.

## List connected numbers

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/accounts \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# -> { "accounts": [ { "phoneNumberId":"...", "businessAccountId":"...", "isActive":true, ... } ] }
```

## Quality & health

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/phone-quality \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"   # quality_rating, throughput, status
curl https://api.eduue.com.br/ext/v1/whatsapp/account-health \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"   # review status, restrictions, healthy
```

## Connect / set active / disconnect (dashboard or JWT plane)

Connecting a number (Embedded Signup / manual) and switching the active one are done in the dashboard (**Conectar WhatsApp**) or on the internal JWT plane `/api/v1/whatsapp/accounts*` (user session). Disconnecting via the dashboard removes the number from this org but **does not** remove it from Meta.

## Migrate a number to another portfolio (deregister)

Meta only allows a number in **one** WhatsApp account. If you get error **#2655122** ("número já registrado") when adding a number elsewhere, you must **deregister** it from the current account first.

- In the dashboard: **Números de telefone → Migrar / remover** (page `/migrar-numero`).
- Programmatically (user session / JWT plane): `POST /api/v1/whatsapp/phone/deregister` with `{ "phoneNumberId": "...", "accessToken": "<optional>" }`.
  - If the number is connected to this org, the stored token is used automatically.
  - If it lives in another portfolio (not connected here), pass an `accessToken` (System User token with `whatsapp_business_management` over the number).

```bash
# from a session-authenticated context (e.g. the eduuw BFF):
curl -X POST https://api.eduue.com.br/api/v1/whatsapp/phone/deregister \
  -H "Authorization: Bearer $SESSION_JWT" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"phoneNumberId":"1216540478200295"}'
```
After success, wait up to ~3 minutes, then add the number to the new portfolio in the Meta panel.

> ⚠️ Deregister is destructive: the number stops sending/receiving on the current account until re-registered. Confirm before running.
