---
name: quick-replies
description: Manage canned quick-reply snippets (with shortcuts) for agents via the eduuw API.
---

# Quick Replies

## When to Use

Managing reusable canned responses (with `/shortcut` triggers) that agents insert in the inbox. Pure CRUD over saved snippets.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. See `eduuw-rules`.

## CRUD

A quick reply has `title`, a `shortcut` (e.g. `/boleto`), and the `text` body. Send `id` to update an existing one; omit it to create.

```bash
# list
curl https://api.eduue.com.br/ext/v1/whatsapp/quick-replies \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "quick_replies":[ {"id":"...","title":"2ª via boleto","shortcut":"/boleto","text":"..."} ], "total": 1 }

# create / update
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/quick-replies \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"title":"Saudação","shortcut":"/oi","text":"Olá! Sou da secretaria, como posso ajudar?"}'

# delete
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/quick-replies/<ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

```js
async function saveQuickReply(qr) {
  return fetch(`${BASE}/quick-replies`, {
    method: 'POST',
    headers: { 'X-API-Key': KEY, 'X-Tenant-ID': TENANT, 'Content-Type': 'application/json' },
    body: JSON.stringify(qr), // { id?, title, shortcut, text }
  })
}
```

## Notes

- Quick replies are agent-side convenience text — sending them to a customer goes through `send-message` / `conversations` (the actual message send).
- `shortcut` should be unique per tenant; use a `/` prefix by convention.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| 403 | Key missing `whatsapp:write` | Mint a key with write scope |
| duplicate shortcut | Shortcut already used | Pick a unique shortcut |
