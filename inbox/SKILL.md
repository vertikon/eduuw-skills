---
name: inbox
description: Team inbox — assign conversations, set status, toggle bot/human control, and add internal notes via the eduuw API.
---

# Inbox (Atendimento)

## When to Use

Building a human-agent inbox on top of WhatsApp conversations: triage conversations by status, assign them to agents, hand off between the bot and a human, and keep private internal notes.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID` (+ `Content-Type: application/json`). See `eduuw-rules`.

The inbox is an **overlay** on the conversations of the connected number — the conversation id is the contact's phone (digits only). Reading the conversations/messages themselves is in the `conversations` endpoints (`/conversations`, `/conversations/{id}/messages`).

## Conversation state

Each conversation has: `status` (`open` | `pending` | `resolved`), `assignedTo` (free-text agent id/name), `control` (`bot` | `human`). No row = defaults (`open` / unassigned / `bot`).

```bash
# list the state overlay of all conversations of the tenant
curl https://api.eduue.com.br/ext/v1/whatsapp/inbox/state \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "states": [ { "conversationId":"5585992009999","status":"open","assignedTo":"","control":"bot","updatedAt":"..." } ] }
```

Update a conversation (send the full desired triple — it upserts):

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/inbox/state/5585992009999 \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"status":"pending","assignedTo":"ana","control":"human"}'
```

```js
// take over from the bot and assign to an agent
await fetch(`https://api.eduue.com.br/ext/v1/whatsapp/inbox/state/${phone}`, {
  method: 'POST',
  headers: { 'X-API-Key': KEY, 'X-Tenant-ID': TENANT, 'Content-Type': 'application/json' },
  body: JSON.stringify({ status: 'pending', assignedTo: 'ana', control: 'human' }),
})
```

**Bot ↔ human handoff:** set `control:"human"` to pause automation for that conversation (the workflow engine respects it — see `automation`), `control:"bot"` to resume. A workflow `handoff` node sets `status:"pending"` + `control:"human"` automatically.

## Internal notes

Notes are private (never sent to the customer):

```bash
# list notes of a conversation
curl https://api.eduue.com.br/ext/v1/whatsapp/inbox/state/5585992009999/notes \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# add a note
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/inbox/state/5585992009999/notes \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"body":"Cliente pediu 2ª via do boleto","author":"ana"}'
```

## Typical flow

1. New inbound arrives → conversation defaults to `open` / `bot`.
2. Bot/automation answers (see `ai-agent`, `automation`).
3. Agent picks it up → `POST /inbox/state/{phone}` `{control:"human", assignedTo:"ana", status:"pending"}`.
4. Agent replies via `POST /conversations/{phone}/messages` (see `send-message` for raw sends).
5. Done → `{status:"resolved"}`.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `tenant não identificado` | Missing `X-Tenant-ID` | Pass the tenant header |
| 401 / 403 | Bad key or tenant not allowed | Check key scopes (`whatsapp:read`/`whatsapp:write`) |
| empty `states` | No conversation has an overlay yet | States are created on first update; conversations still exist in `/conversations` |
