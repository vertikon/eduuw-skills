---
name: whatsapp-flows
description: Manage WhatsApp Flows (native in-chat forms) — create, upload definition, publish via the eduuw API.
---

# WhatsApp Flows

## When to Use

Building native in-chat **forms** (WhatsApp Flows): multi-screen data collection rendered inside WhatsApp (e.g. lead capture, booking). This manages the Flow lifecycle on Meta. Not to be confused with `automation` (conversational workflows) — a Flow is a Meta form you can send/trigger from a message.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. See `eduuw-rules`.

## Lifecycle

`create (DRAFT)` → `upload flow.json (assets)` → `publish`. Deleting deprecates the Flow.

```bash
# list
curl https://api.eduue.com.br/ext/v1/whatsapp/flows \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# create a DRAFT
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/flows \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"name":"Lead curso","categories":["LEAD_GENERATION"]}'
# → { "id":"<FLOW_ID>", "status":"DRAFT" }

# upload the flow.json definition (screens/components)
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/flows/<FLOW_ID>/assets \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"flow_json": { /* Meta Flow JSON */ }}'

# publish (makes it usable in messages)
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/flows/<FLOW_ID>/publish \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# inspect / deprecate
curl https://api.eduue.com.br/ext/v1/whatsapp/flows/<FLOW_ID> -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/flows/<FLOW_ID> -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Notes

- Flows are encrypted by Meta — the connected number must have Flows encryption configured (set during onboarding).
- After publishing, send the Flow via an interactive message (`send-message`) or wire its submit to a `functions`/`webhook-setup` handler to receive responses.
- `flow.json` follows Meta's Flow JSON schema (screens, components, data-exchange endpoint).

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `424 no_whatsapp_number_connected` | No connected number | Connect a number (`onboarding`) |
| publish fails — encryption | Flows public key not uploaded | Finish Flows encryption setup on the number |
| invalid flow_json | Schema error | Validate against Meta's Flow JSON schema |
