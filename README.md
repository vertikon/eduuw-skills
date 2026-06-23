# eduuw-skills

Agent skills for the **eduuw** WhatsApp Business API ([dashboard.eduuw.com.br](https://dashboard.eduuw.com.br)).

Give your local AI agent context about the eduuw API — how to send messages, manage templates, broadcasts, contacts, webhooks, serverless functions and the AI bot, over the HTTP edge (`/ext/v1/whatsapp`).

There are **two ways** to give an agent this capability — pick one (or both):

| | **Skills** (this repo) | **MCP** (live tools) |
|---|---|---|
| What the agent gets | Markdown context it reads on demand | Callable tools it invokes directly |
| Setup | `npx skills add` (folders in `.agents/skills/`) | one `claude mcp add` (no files) |
| Best when | You want the agent to *write code* against the API | You want the agent to *operate* the API itself |
| Discovery | `eduuw-rules` loads first, the rest on demand | `ListTools`, filtered by your key's scopes |

## Install (skills)

```bash
npx skills add vertikon/eduuw-skills
```

This drops the `SKILL.md` files into your project's `.agents/skills/`. Your agent loads `eduuw-rules` first (auth, base URL, conventions) and the rest on demand. One folder per skill is the standard layout and is optimal for on-demand agents — only the relevant skill is loaded, not all 20 at once.

## MCP (centralized — all operations as tools)

The eduue edge exposes **every** operation as an MCP tool (curated from the OpenAPI `x-mcp` annotations). One command and the agent can call the API directly — no per-skill folders:

```bash
claude mcp add --transport http eduuw \
  https://api.eduue.com.br/mcp \
  --header "X-API-Key: $EDUUE_API_KEY" \
  --header "X-Tenant-ID: $TENANT_ID"
```

- **Transport:** Streamable HTTP at `https://api.eduue.com.br/mcp`.
- **Auth:** the same M2M credential as the skills — `X-API-Key` (prefix `vtk_live_`) + `X-Tenant-ID`. Required on **every** request (revocation takes effect immediately).
- **Scope-filtered:** `ListTools` only returns tools your key is allowed to call. **Mint a key with the minimum scopes** (e.g. `whatsapp:read` + `whatsapp:write`) so the agent's surface stays small and safe — a broad key also exposes LMS/analytics/engagement tools beyond WhatsApp.
- The MCP tools mirror these skills (e.g. `whatsapp_list_templates`, `whatsapp_create_broadcast`, `whatsapp_list_contacts`, `whatsapp_save_ai_bot`).

### Tool surface (with a `whatsapp:*` key)

| Group | Tools |
|-------|-------|
| Send & content | `whatsapp_create_template`, `whatsapp_generate_template`, `whatsapp_list_templates`, `whatsapp_create_broadcast`, `whatsapp_list_broadcasts`, `whatsapp_send_interactive`, `whatsapp_get_message_status`, `wa_marketing_dispatch` |
| Inbox & bot | `whatsapp_cx_insights`, `whatsapp_cx_topics`, `whatsapp_list_autoreply`, `whatsapp_save_autoreply`, `whatsapp_list_quick_replies`, `whatsapp_save_quick_reply`, `whatsapp_save_routing`, `whatsapp_save_ai_bot`, `whatsapp_train_ai_bot` |
| Contacts & consent | `whatsapp_list_contacts`, `whatsapp_list_optins`, `whatsapp_set_optin` |
| Numbers & health | `whatsapp_list_accounts`, `whatsapp_account_health`, `whatsapp_phone_quality` |
| Flows, functions, catalog | `whatsapp_create_flow`, `whatsapp_list_flows`, `whatsapp_list_functions`, `whatsapp_list_catalog` |
| Analytics & cost | `whatsapp_messaging_analytics`, `whatsapp_conversation_analytics`, `whatsapp_cost_by_category`, `whatsapp_ctwa_attribution`, `whatsapp_dispatch_get_job` |

> A broader key (LMS/education scopes) additionally exposes `analytics_*` (LMS KPIs, courses, student-360, at-risk), `neuralead_*` (engagement orbits / dropout E-score), `rewards_redeem`, and `student_success_*`. Some of these **act on production** (mass dispatch, point redemption) — scope down unless you need them.

> ⚠️ Tools run with full agent permissions against the **live, multi-tenant** edge. Several are destructive or mass-acting (broadcasts, dispatch, redeem, deregister). Use a scoped key, and prefer read tools when exploring.

## Skills

| Skill | Purpose |
|-------|---------|
| `eduuw-rules` | Foundational context: base URL, auth (`X-API-Key` + `X-Tenant-ID`), scopes, 24h window, business rules. Always loaded. |
| `onboarding` | Self-service setup links + connection status (customer connects their own WABA). |
| `phone-numbers` | Connected numbers, quality/health, set active, deregister/migrate. |
| `send-message` | Send text, template, media and interactive messages; status & mark-read. |
| `whatsapp-templates` | List/create/delete templates; AI-generate a template. |
| `conversations` | Read conversation threads + message history; reply as agent. |
| `inbox` | Team inbox: assign, status, bot-human handoff, internal notes. |
| `routing` | Auto-assign inbound conversations to agents (round-robin/fixed). |
| `quick-replies` | Canned reply snippets with shortcuts. |
| `broadcast-campaign` | Bulk template sends to a list/segment (opt-out aware). |
| `contacts-management` | Contacts, tags/segments, opt-in/out consent. |
| `automation` | Node-based workflows that reply to inbound (build/simulate/run). |
| `ai-agent` | Configure the AI bot (RAG) and keyword auto-replies. |
| `functions` | Serverless functions (TypeScript reacting to events). |
| `whatsapp-flows` | Native in-chat forms (WhatsApp Flows): create/upload/publish. |
| `webhook-setup` | Subscribe to events with HMAC-signed deliveries. |
| `groups` | WhatsApp groups: members, roles, invite links, join requests. |
| `analytics` | Messaging/conversation analytics, delivery rates, CSV export. |
| `cx-insights` | Agent metrics + AI-classified conversation topics. |
| `billing` | Usage metering + per-tenant plan limits (quota). |

## Auth

Create an API key in the dashboard (**Chaves de API**) and copy your organization id from **Contas** (`X-Tenant-ID`).

```bash
export EDUUE_API_KEY=vtk_live_xxx
export TENANT_ID=<your-org-uuid>
```

The same key + tenant authenticate both the skills' HTTP examples and the MCP server.

## License

MIT © Vertikon / eduuw
