---
name: eduuw-rules
description: Foundational context for the eduuw WhatsApp Business API. Always loaded.
---

# eduuw API Context

eduuw is a multi-tenant WhatsApp Business platform built on the eduue "borda" (external gateway). One HTTP API to send WhatsApp messages, manage templates, broadcasts, contacts, webhooks, serverless functions and an AI bot — per organization (tenant).

There is no language SDK: integrate over plain HTTP (curl / fetch / requests). Every example below works against the live edge.

## Base URL & Auth

- **Base URL:** `https://api.eduue.com.br/ext/v1/whatsapp`
- **Auth headers (every request):**
  - `X-API-Key: $EDUUE_API_KEY` — machine-to-machine key, prefix `vtk_live_` (prod) / `vtk_test_`.
  - `X-Tenant-ID: $TENANT_ID` — the organization (UUID). Required when the key is global; optional when the key is bound to a single tenant.
  - `Content-Type: application/json` on POST/PUT/PATCH.
- Get the key + tenant id in the dashboard: **Contas** (X-Tenant-ID) and **Chaves de API** (create/list keys). Each subconta is its own tenant.

```bash
export EDUUE_API_KEY=vtk_live_xxx
export TENANT_ID=4f249adb-3de8-47bc-8b9b-785cf4181939
```

## Scopes

Keys are scoped. The relevant scopes for this API:
- `whatsapp:read` — list templates/contacts/accounts/functions, read status, analytics.
- `whatsapp:write` — send messages, create templates/broadcasts/contacts/functions.
- `analytics:read` — CX insights.
Mint a key in **Chaves de API** with only the scopes you need.

## Core Conventions

- **Phone numbers:** digits only, country code first, no `+` (ex.: `5585992009999`). The platform normalizes; prefer full E.164 digits.
- **Tenant isolation:** every resource is scoped to `X-Tenant-ID`. A key can only act on tenants it is allowed to.
- **List responses:** objects like `{ "templates": [...] }`, `{ "contacts": [...] }`, `{ "accounts": [...] }`, `{ "functions": [...] }`.
- **Errors:** non-2xx returns `{ "error": "<message>" }`. 401 = bad/missing key or tenant; 403 = tenant not allowed for key; 400 = bad payload.

## Key Business Rules

1. **WhatsApp 24h window:** free-form text only works inside an open conversation (the user messaged you in the last 24h). To start a conversation, send an **approved template** (`messages/template`). See `send-message` and `whatsapp-templates`.
2. **Templates need approval:** create via `templates`, Meta reviews (PENDING → APPROVED). Only APPROVED templates can be sent.
3. **Opt-out is enforced:** contacts with `OPTED_OUT` status are skipped by broadcasts. Manage via `optins`. See `contacts-management`.
4. **One active number per tenant:** a tenant can connect several numbers but one is "active" (used by default). See `phone-numbers`.
5. **A number lives in one WhatsApp account:** to move a number to another portfolio, deregister it first (Meta error #2655122). See `phone-numbers`.

## MCP Server (optional, for AI assistants)

The eduue edge exposes its operations as MCP tools (curated via the OpenAPI `x-mcp` annotations). Point an MCP-capable assistant at the eduue MCP endpoint with the same `X-API-Key`; ListTools is filtered by the key's scopes. Tools mirror the endpoints in these skills (e.g. `whatsapp_list_accounts`, `whatsapp_generate_template`, `whatsapp_list_contacts`, `whatsapp_list_functions`).

## Quick smoke test

```bash
curl -s https://api.eduue.com.br/ext/v1/whatsapp/templates \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```
401 → check key/tenant. 200 with `{"templates":[...]}` → you're connected.

## Skill map

| Skill | Use for |
|-------|---------|
| `send-message` | Send text, template, media, interactive, location, contact-card messages |
| `whatsapp-templates` | List/create/delete templates; generate a template with AI |
| `broadcast-campaign` | Bulk sends to a list/segment (respecting opt-out) |
| `contacts-management` | Contacts + tags/segments + opt-in/out |
| `phone-numbers` | Connected numbers, quality/health, set active, deregister/migrate |
| `webhook-setup` | Subscribe to events (HMAC-signed deliveries) |
| `functions` | Serverless functions (TypeScript) reacting to events |
| `ai-agent` | AI bot / auto-reply over inbound conversations |
