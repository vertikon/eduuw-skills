---
name: functions
description: Manage eduuw serverless functions — TypeScript that reacts to events.
---

# Functions

## When to Use

Author and manage serverless functions: TypeScript that reacts to eduuw events (a message arrives, a broadcast is sent, etc.). Base + auth: see `eduuw-rules`.

> **Status:** function **management** (code + trigger + status) is live. The **execution runtime** runs each function as an **isolated Cloudflare Worker** (Workers for Platforms — see *Execution & isolation*). The dispatcher is online and routing is proven; the deploy→execute wiring is being connected. Until fully wired, a `DEPLOYED` function is stored + routable but may not yet auto-run on events.

## Model

`FunctionDef = { id, name, description, trigger, code, status }`.
- **trigger** (event): `message.received | message.status | conversation.started | broadcast.sent`.
- **status:** `DRAFT | DEPLOYED`.
- **code:** TypeScript with a default async handler `export default async function onMessage(event, ctx) { ... }`.

## List

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/functions \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# -> { "functions": [ { "id":"...", "name":"echo", "trigger":"message.received", "status":"DEPLOYED" } ] }
```

## Create / update (id empty = create) and deploy

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/functions \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{
    "name": "echo",
    "description": "Responde a qualquer mensagem com o mesmo texto",
    "trigger": "message.received",
    "status": "DEPLOYED",
    "code": "export default async function onMessage(event, ctx) {\n  await ctx.wa.sendText(event.message.from, `Você disse: ${event.message.text}`)\n}"
  }'
# -> { "id": "<FUNCTION_ID>" }
```
Send `status:"DRAFT"` to save without deploying. Include the `id` to update an existing function.

## Get (with code) / delete

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/functions/<ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/functions/<ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Handler shape (target runtime)

```typescript
// event: the triggering event (e.g. { message: { from, text } })
// ctx:   helpers — ctx.wa.sendText(to, text), ctx.ai.chat({system,user}), ctx.fetch(url, init), ctx.secrets.X
export default async function onMessage(event, ctx) {
  const reply = await ctx.ai.chat({ system: 'Você é um atendente.', user: event.message.text })
  await ctx.wa.sendText(event.message.from, reply)
}
```

## Execution & isolation

Functions run as **isolated Cloudflare Workers** (Workers for Platforms), not `eval` inside the platform process:

- Each function is its **own user worker** inside a dispatch namespace — no shared state, with per-worker CPU/memory/time limits.
- A front **dispatcher worker** authenticates the request (shared secret) and routes to the right user worker by script name (`X-Function-Script`).
- Strong multi-tenant isolation: a function only ever sees its own `ctx` (`wa`/`ai`/`fetch`/`secrets`) — never another tenant's data.
- `status:"DEPLOYED"` publishes the code as a user worker; the runtime invokes it on the `trigger` event with `(event, ctx)`. **Secrets are injected per-worker — never put secrets in the `code`.**

## Starter templates

The dashboard (**Funções**) ships ready-made examples you can clone: minimal echo bot, restaurant-reservation agent, school↔parents assistant, e-commerce order assistant. Start from one, then customize and add secrets.

## Tip

For no-code automations that react to events without writing TypeScript, see `ai-agent` (AI bot + keyword auto-reply).
