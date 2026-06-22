---
name: send-message
description: Send WhatsApp messages (text, template, media, interactive) via the eduuw API.
---

# Send Message

## When to Use

Building code to send a WhatsApp message through eduuw. Covers message types, the 24h window rule, and status.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID` (+ `Content-Type: application/json`). See `eduuw-rules`.

## The 24h window (read first)

- **Inside an open window** (the user messaged you in the last 24h) → you may send free-form **text/media/interactive**.
- **Outside the window** (cold start) → you may only send an **approved template** (`messages/template`). A free-form text outside the window fails. See `whatsapp-templates`.

## Text

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/messages/text \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" \
  -H "Content-Type: application/json" \
  -d '{"to":"5585992009999","content":"Olá! Como posso ajudar?"}'
```

```js
await fetch('https://api.eduue.com.br/ext/v1/whatsapp/messages/text', {
  method: 'POST',
  headers: { 'X-API-Key': KEY, 'X-Tenant-ID': TENANT, 'Content-Type': 'application/json' },
  body: JSON.stringify({ to: '5585992009999', content: 'Olá!' }),
})
```

```python
import requests
requests.post('https://api.eduue.com.br/ext/v1/whatsapp/messages/text',
  headers={'X-API-Key': KEY, 'X-Tenant-ID': TENANT},
  json={'to': '5585992009999', 'content': 'Olá!'})
```

## Template (start a conversation outside the window)

`template_name` must be APPROVED. `parameters` fills the `{{1}}`,`{{2}}`… body variables.

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/messages/template \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"to":"5585992009999","template_name":"confirmacao_matricula","language":"pt_BR","parameters":{"1":"João","2":"Enfermagem"}}'
```

## Media

Upload first via `POST /media/upload` (multipart), then send by `media_id`:

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/messages/media \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"to":"5585992009999","type":"document","media_id":"<MEDIA_ID>","caption":"Seu boleto"}'
```
`type`: `image | video | document | audio`.

## Interactive (buttons / list)

`interactive` is the raw Meta Cloud API interactive object (`type: button` or `type: list` + `action`):

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/messages/interactive \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"to":"5585992009999","interactive":{"type":"button","body":{"text":"Confirma a presença?"},"action":{"buttons":[{"type":"reply","reply":{"id":"yes","title":"Sim"}},{"type":"reply","reply":{"id":"no","title":"Não"}}]}}}'
```
Constraints (Meta): button title ≤ 20 chars, ≤ 3 buttons; list rows ≤ 10/section.

## Status & mark-read

```bash
# delivery/read status of a sent message
curl https://api.eduue.com.br/ext/v1/whatsapp/messages/<MESSAGE_ID>/status \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# mark an inbound message as read
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/messages/<MESSAGE_ID>/read \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `account not found for tenant` | No connected/active number | Connect a number (`phone-numbers`) |
| 24h window failure from Meta | Free-form outside window | Send an approved template |
| template not approved | Template still PENDING/REJECTED | Wait for approval / fix the template |
| 401 / 403 | Bad key or tenant not allowed | Check `X-API-Key` + `X-Tenant-ID` scopes |
