---
name: ai-agent
description: Configure the eduuw AI bot and keyword auto-replies over inbound WhatsApp.
---

# AI Agent

## When to Use

Make eduuw answer inbound WhatsApp messages automatically — either with an **AI bot** (LLM over your knowledge) or with **keyword auto-replies** (deterministic). Base + auth: see `eduuw-rules`.

These run on the server (no code), unlike `functions` (custom TypeScript).

## AI bot

A per-tenant autonomous bot that replies to inbound messages using an LLM, optionally grounded on trained sources (RAG). Inert until enabled and (for RAG) trained.

### Read / configure

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/ai-bot \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/ai-bot \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "system_prompt": "Você é o atendente da escola. Responda sobre cursos, matrícula e horários.",
    "destino": "curso-enfermagem",
    "fallback_text": "Um momento, vou chamar um atendente."
  }'
```
Fields: `enabled`, `system_prompt`, `agent_key` (optional named agent), `destino` (RAG knowledge scope), `fallback_text`.

### Train (RAG sources)

```bash
# index text or a URL as knowledge for the bot
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/ai-bot/train \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"text":"A matrícula abre em julho. Mensalidade R$ 199...", "destino":"curso-enfermagem"}'

# list trained sources
curl https://api.eduue.com.br/ext/v1/whatsapp/ai-bot/sources \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Keyword auto-reply (deterministic)

Rule-based replies matched against inbound text — runs before/instead of the AI bot for known intents.

```bash
# list
curl https://api.eduue.com.br/ext/v1/whatsapp/autoreply \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# create/update a rule
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/autoreply \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"name":"saudação","match_type":"CONTAINS","keyword":"oi","reply_text":"Olá! Como posso ajudar?","enabled":true,"priority":10}'

# delete a rule
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/autoreply/<ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```
`match_type`: `CONTAINS | EQUALS | STARTS`. Higher `priority` wins when multiple rules match.

## Conversation insights (analytics)

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/cx/insights -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl https://api.eduue.com.br/ext/v1/whatsapp/cx/topics  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```
Scope: `analytics:read`. Topics are AI-classified from recent conversations.

## Choosing

- **Known intents / compliance-sensitive replies** → `autoreply`.
- **Open-ended Q&A grounded on your content** → AI bot (train it).
- **Custom logic / external API calls** → `functions` (TypeScript).
