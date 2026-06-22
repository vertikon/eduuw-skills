---
name: cx-insights
description: Customer-experience insights — agent metrics and AI-classified conversation topics via the eduuw API.
---

# CX Insights

## When to Use

Reading service-quality insights over the inbox: per-operator metrics, overall attendance overview, and the topics customers are talking about (classified by AI). Read-only analytics for managers/dashboards.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. Scope `analytics:read` (CX) — mint a key with it in **Chaves de API**. See `eduuw-rules`.

## Attendance metrics (operators + overview)

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/cx/insights \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → métricas por operador + visão geral do atendimento (volume, resolvidas, etc.)
```

## Conversation topics (AI-classified)

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/cx/topics \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → tópicos das conversas agrupados/classificados por IA (ex.: matrícula, financeiro, dúvida)
```

```js
const insights = await (await fetch(`${BASE}/cx/insights`, { headers })).json()
const topics   = await (await fetch(`${BASE}/cx/topics`,   { headers })).json()
```

## Notes

- These complement the raw `analytics` (volumes/delivery) and `billing` (usage/quota) with a service-quality lens.
- Topic classification uses the platform AI; results improve as more conversations accumulate.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| 403 | Key missing `analytics:read` | Mint a key with the `analytics:read` scope |
| empty result | No conversations yet this period | Insights appear after attendance traffic exists |
