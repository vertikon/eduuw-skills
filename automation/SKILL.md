---
name: automation
description: Conversational automation — build, simulate and run node-based workflows that reply to inbound WhatsApp messages via the eduuw API.
---

# Automation (Workflows)

## When to Use

Building automated conversational flows that react to inbound messages: send text/templates/buttons, wait for replies, branch on keywords, answer with AI, and hand off to a human.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID` (+ `Content-Type: application/json`). See `eduuw-rules`.

## How it runs

A workflow is a **graph of nodes**. When an inbound message arrives, the engine matches an **active** workflow by its trigger (keyword and/or number), walks the graph sending **real** messages, **pauses** at a `wait` node (durable state per conversation) and **resumes** on the next inbound. No active match → nothing happens (the bot/auto-reply still apply). Manual control (`control:"human"` in `inbox`) pauses automation for that conversation.

## Node types

| `type` | Config fields | Does |
|--------|---------------|------|
| `start` | `next` | Entry point |
| `send_text` | `text`, `next` | Sends a text |
| `send_template` | `template`, `language`, `next` | Sends an approved template |
| `send_interactive` | `text`, `buttons:[{title}]` (≤3), `next` | Sends quick-reply buttons |
| `agent` | `agentKey`, `destino`, `prompt`, `text` (fallback), `next` | AI reply: trained agent (RAG by `agentKey`+`destino`) → GLM (`prompt`) → `text` fallback |
| `wait` | `next` | Pauses; resumes at `next` on the next inbound |
| `decide` | `branches:[{keyword,next}]` | Routes by keyword (empty keyword = default) |
| `handoff` | — | Transfers to a human (sets the inbox to pending/human) |

## CRUD

```bash
# list
curl https://api.eduue.com.br/ext/v1/whatsapp/workflows \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# create (status draft|active|paused; only `active` runs live)
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/workflows \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{
    "name":"Atendimento matrícula",
    "status":"active",
    "triggerKeyword":"matricula",
    "definition":{"nodes":[
      {"id":"start","type":"start","next":"n1"},
      {"id":"n1","type":"send_text","text":"Olá! Vou te ajudar com a matrícula.","next":"n2"},
      {"id":"n2","type":"wait","next":"n3"},
      {"id":"n3","type":"agent","agentKey":"atendente-matricula","destino":"curso-enfermagem","text":"Um momento, vou te transferir.","next":"end"}
    ]}
  }'
```

`GET|PUT|DELETE /workflows/{id}` manage one workflow.

## Simulate (no real send)

Validate the graph deterministically before activating:

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/workflows/<WF_ID>/simulate \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"inputs":["oi","matricula"]}'
# → { "trace": [ {"node":"n1","type":"send_text","detail":"Olá! ..."}, ... ] }
```

`inputs[0]` is the triggering message; each later input is consumed by a `wait`.

## Rules

1. Only `status:"active"` workflows run live. `draft`/`paused` are inert (still simulatable).
2. The `agent` node needs a trained agent (`agentKey`, see `ai-agent`) for RAG answers; without it, it uses the node `prompt` via GLM, else the `text` fallback.
3. Quota: if the tenant's plan limits `msg_out`/`ai_call` are hit, the engine skips that action (see `billing`).
4. Trigger by `triggerKeyword` (inbound contains it) and/or `triggerPhoneNumberId`; empty keyword = any message.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `workflow não encontrado` | Bad id / wrong tenant | Check the id and `X-Tenant-ID` |
| `definição do workflow inválida` | `definition` is not valid JSON | Fix the `nodes` JSON |
| Workflow never fires | Not `active`, or trigger doesn't match | Set `status:"active"`, check `triggerKeyword` |
