# eduuw-skills

Agent skills for the **eduuw** WhatsApp Business API ([dashboard.eduuw.com.br](https://dashboard.eduuw.com.br)).

Give your local AI agent context about the eduuw API — how to send messages, manage templates, broadcasts, contacts, webhooks, serverless functions and the AI bot, over the HTTP edge (`/ext/v1/whatsapp`).

## Install

```bash
npx skills add vertikon/eduuw-skills
```

This drops the `SKILL.md` files into your project's `.agents/skills/`. Your agent loads `eduuw-rules` first (auth, base URL, conventions) and the rest on demand.

Prefer MCP? The eduue edge exposes the same operations as MCP tools:

```bash
claude mcp add --transport http eduuw \
  https://api.eduue.com.br/mcp \
  --header "X-API-Key: $EDUUE_API_KEY" \
  --header "X-Tenant-ID: $TENANT_ID"
```

## Skills

| Skill | Purpose |
|-------|---------|
| `eduuw-rules` | Foundational context: base URL, auth (`X-API-Key` + `X-Tenant-ID`), scopes, 24h window, business rules. Always loaded. |
| `send-message` | Send text, template, media and interactive WhatsApp messages; status & mark-read. |
| `whatsapp-templates` | List/create/delete templates; AI-generate a template. |
| `broadcast-campaign` | Bulk template sends to a list/segment (opt-out aware). |
| `contacts-management` | Contacts, tags/segments, opt-in/out consent. |
| `phone-numbers` | Connected numbers, quality/health, set active, deregister/migrate. |
| `webhook-setup` | Subscribe to events with HMAC-signed deliveries. |
| `functions` | Manage serverless functions (TypeScript reacting to events). |
| `ai-agent` | Configure the AI bot (RAG) and keyword auto-replies. |

## Auth

Create an API key in the dashboard (**Chaves de API**) and copy your organization id from **Contas** (`X-Tenant-ID`).

```bash
export EDUUE_API_KEY=vtk_live_xxx
export TENANT_ID=<your-org-uuid>
```

## License

MIT © Vertikon / eduuw
