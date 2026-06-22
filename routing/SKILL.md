---
name: routing
description: Configure automatic assignment of inbound conversations to agents via the eduuw API.
---

# Routing (Roteamento automático)

## When to Use

Configuring how new inbound conversations are automatically assigned to agents — round-robin across a team, or fixed to one agent. Works with the `inbox` (the assignment it writes) and human handoff.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. See `eduuw-rules`.

## Read / save config

Fields: `enabled` (bool), `strategy` (`round_robin` | `fixed`), `agents` (list of agent ids for round-robin), `fixed` (agent id when strategy is fixed).

```bash
# read
curl https://api.eduue.com.br/ext/v1/whatsapp/routing \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# save (round-robin over a team)
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/routing \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"enabled":true,"strategy":"round_robin","agents":["ana","bruno","carla"],"fixed":""}'

# save (everything to one agent)
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/routing \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"enabled":true,"strategy":"fixed","agents":[],"fixed":"ana"}'
```

## Notes

- When `enabled`, a new conversation gets an `assignedTo` automatically (visible via `inbox` → `/inbox/state`).
- Routing assigns; it does not force `control:"human"`. Combine with `automation`/`ai-agent` so the bot answers until a human takes over.
- `enabled:false` leaves conversations unassigned (manual pickup in the inbox).

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| 403 | Key missing `whatsapp:write` | Mint a key with write scope |
| assignments not happening | `enabled:false` or empty `agents` | Enable and provide agent ids |
