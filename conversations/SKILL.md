---
name: conversations
description: Read conversation threads and message history, and reply as an agent, via the eduuw API.
---

# Conversations

## When to Use

Reading the conversation list and message history of the connected number, and sending a reply inside an open conversation (agent panel). For the assignment/status overlay (who's handling it, bot↔human) see `inbox`; for cold-start sends and message types see `send-message`.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. See `eduuw-rules`.

## List conversations

```bash
curl "https://api.eduue.com.br/ext/v1/whatsapp/conversations?limit=50&offset=0" \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "conversations":[ {"id":"5585992009999","contactName":"João","lastMessage":"...","updatedAt":"...","unread":2} ], "total": 12 }
```

The conversation `id` is the contact's phone (digits only).

## Read one conversation + messages

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/conversations/5585992009999 \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl "https://api.eduue.com.br/ext/v1/whatsapp/conversations/5585992009999/messages?limit=50&offset=0" \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "messages":[ {"id":"wamid...","direction":"inbound","content":"oi","createdAt":"..."} ], "total": 40 }
```

`direction`: `inbound` (from the customer) | `outbound` (sent by you).

## Reply as agent

Sends inside the open 24h window (free-form). Outside the window you must use a template (`send-message`).

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/conversations/5585992009999/messages \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"content":"Oi! Já estou verificando sua matrícula."}'
```

```js
async function reply(phone, text) {
  return fetch(`https://api.eduue.com.br/ext/v1/whatsapp/conversations/${phone}/messages`, {
    method: 'POST',
    headers: { 'X-API-Key': KEY, 'X-Tenant-ID': TENANT, 'Content-Type': 'application/json' },
    body: JSON.stringify({ content: text }),
  })
}
```

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `account not found for tenant` | No connected number | Connect a number (`onboarding`/`phone-numbers`) |
| 24h window failure | Free-form outside the window | Send an approved template (`send-message`) |
| empty `conversations` | No inbound yet | Conversations appear after the first inbound message |
