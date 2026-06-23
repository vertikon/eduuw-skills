---
name: contacts-management
description: Manage WhatsApp contacts, tags/segments and opt-in/out consent in eduuw.
---

# Contacts Management

## When to Use

Create/update/list contacts, organize them into segments (tags), and manage opt-in/out consent. Base + auth: see `eduuw-rules`.

## Contact model

A contact is `{ id, phone, name, tags[] }` scoped to the tenant. A **segment** is simply the set of contacts sharing a `tag`.

## List

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/contacts \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# -> { "contacts": [ { "id": "...", "phone": "5585...", "name": "João", "tags": ["leads","curso-x"] } ] }
```

## Upsert (single or batch)

Upsert is by `phone` (creates or updates name/tags). Use the array form for imports.

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/contacts \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"contacts":[
        {"phone":"5585992009999","name":"João","tags":["leads","curso-enfermagem"]},
        {"phone":"5511988887777","name":"Maria","tags":["alunos"]}
      ]}'
# -> { "upserted": 2 }
```
Tagging a contact = upsert it with the new `tags` array (tags replace, not merge).

## Delete

```bash
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/contacts/<PHONE> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

> ⚠️ Permanent: removes the contact and its tags for the tenant. To stop messaging someone without losing the record, set `OPTED_OUT` (below) instead of deleting.

## Segments

There is no separate segments endpoint — derive them client-side from contact tags:

```js
const { contacts } = await get('/contacts')
const segments = {}
for (const c of contacts) for (const t of c.tags || []) segments[t] = (segments[t]||0)+1
// segments = { leads: 12, alunos: 40, ... }
```
To broadcast a segment, collect the phones of contacts with that tag → `broadcast-campaign`.

## Opt-in / opt-out (consent)

Broadcasts skip `OPTED_OUT` contacts. Register/read consent:

```bash
# set status
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/optins \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"phone":"5585992009999","status":"OPTED_OUT","source":"manual","reason":"pediu para sair"}'

# list (e.g. all opt-outs)
curl "https://api.eduue.com.br/ext/v1/whatsapp/optins?status=OPTED_OUT&limit=5000" \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```
`status`: `OPTED_IN | OPTED_OUT`.

## Importing from WhatsApp

You can seed contacts from existing WhatsApp data:
- **Groups:** `GET /groups` then `GET /groups/{id}/members` → upsert members.
- **Conversations (inbound):** `GET /conversations` → upsert senders (these already opted in by messaging you).
