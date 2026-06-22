---
name: onboarding
description: Self-service WhatsApp onboarding — generate setup links so a customer connects their own WABA, and read connection status via the eduuw API.
---

# Onboarding (Conectar número)

## When to Use

Letting a customer (sub-account / tenant) connect their **own** WhatsApp Business number — either by generating a self-service link they open themselves (no dashboard login), or by reading the connection status of a tenant. For listing/activating/migrating already-connected numbers, see `phone-numbers`.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID` (+ `Content-Type: application/json`). See `eduuw-rules`.

## First-run checklist (start here)

One call to drive a guided onboarding UI/agent — what's done, what's blocking, and the next step:

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/onboarding/status \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "connected":false,"hasNumber":false,"webhookConfigured":false,"hasTemplate":false,
#     "blockers":["Conecte um número WhatsApp"], "nextStep":"connect_number" }
```

`nextStep`: `connect_number` → `create_template` → `ready`. Loop on it until `ready`.

## Connection status

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/account \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "connected":true, "businessAccountId":"...", "phoneNumberId":"...",
#     "kind":"production", "isCoexistence":false, "webhookConfigured":true,
#     "blockers":[] }
```

`kind`: `production` | `sandbox` (a test number provided by the platform). `blockers[]` lists what still prevents going live (no token, no number, webhook not configured).

## Setup links (self-service)

Generate a single-use link (expires in 7 days) and send it to the customer. They open it, log into **their** Facebook, pick/create the WhatsApp Business account + number via Meta Embedded Signup — the token flows back to the platform. No platform login needed.

```bash
# generate a link for a tenant
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/setup-links \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"tenantId":"'"$TENANT_ID"'","label":"Escola Aurora","ttlHours":168}'
# → { "token":"<hex>", "expiresAt":"2026-06-29T...", "status":"active" }

# list links of the tenant
curl https://api.eduue.com.br/ext/v1/whatsapp/setup-links \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

Build the customer URL from the token: `https://<app-host>/whatsapp-setup/<token>` (the public page renders the Meta Embedded Signup and finishes the connection — no auth).

## Embedded Signup config (for a custom UI)

If you render the Meta "Connect" button yourself, read the app config (never exposes the app secret):

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/embedded-signup/config \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "appId":"...", "configId":"...", "graphVersion":"v22.0",
#     "configured":true, "sandboxAvailable":false }
```

`configured:false` → the platform hasn't set the Meta app credentials yet (1-click connect unavailable); fall back to manual onboarding in the dashboard.

## Rules

1. A number lives in **one** WhatsApp account — to migrate it, deregister first (see `phone-numbers`).
2. Setup links are **single-use** and expire (default 7 days); generate a new one if used/expired.
3. A tenant can connect several numbers but one is **active** (see `phone-numbers`).
4. `kind:"sandbox"` lets a customer test without their own Meta number — not for production traffic.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `Embedded Signup não configurado` | Platform missing `META_APP_ID/SECRET/ES_CONFIG_ID` | Configure on the server, or onboard manually |
| `link inválido: expirado/já utilizado` | Setup link consumed or past TTL | Generate a new link |
| `connected:false` with blockers | Number not fully connected | Resolve the listed `blockers[]` |
