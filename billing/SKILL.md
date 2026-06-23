---
name: billing
description: Usage metering and plan limits — read per-tenant consumption and set monthly quotas via the eduuw API.
---

# Billing (Uso & Planos)

## When to Use

Reading how much a tenant consumed this month (messages, automation runs, AI usage) and setting per-tenant plan limits to bill by usage or cap spend.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID` (+ `Content-Type: application/json`). See `eduuw-rules`.

## Metrics

The engine meters per tenant per month:

| Metric | Counts |
|--------|--------|
| `msg_in` | Inbound messages received |
| `msg_out` | Messages sent by the automation engine |
| `wf_run` | Workflows triggered |
| `ai_call` | AI agent calls |
| `ai_tokens` | GLM tokens consumed |

> Scope today: automation-driven traffic + inbound. Manual sends (broadcast/raw) are not yet metered.

## Read usage

```bash
# current month (or ?period=YYYY-MM)
curl https://api.eduue.com.br/ext/v1/whatsapp/usage \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "period":"2026-06",
#     "totals":{"msg_in":120,"msg_out":98,"wf_run":40,"ai_call":35,"ai_tokens":18450},
#     "limits":{"msg_out":1000,"ai_call":500,"ai_tokens":0},
#     "planName":"Pro" }
```

## Plan & limits

A plan sets monthly limits. **`0` = unlimited.** No plan row = everything unlimited (no enforcement).

```bash
# read the plan
curl https://api.eduue.com.br/ext/v1/whatsapp/plan \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# set the plan (only msg_out / ai_call / ai_tokens are enforced)
curl -X PUT https://api.eduue.com.br/ext/v1/whatsapp/plan \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"planName":"Pro","limitMsgOut":100000,"limitAiCall":5000,"limitAiTokens":0}'
```

> ⚠️ Enforcement is immediate. A low `limitMsgOut`/`limitAiCall` makes the automation engine **start skipping sends/AI for that tenant** as soon as the month's counter passes it. Use `0` (unlimited) unless you intend to cap. PUT replaces the whole plan row.

```js
// gate provisioning by usage
const u = await (await fetch(`${BASE}/usage`, { headers })).json()
if (u.limits.msg_out && u.totals.msg_out >= u.limits.msg_out) {
  // over quota — the engine already skips sends; warn the customer / upsell
}
```

## Enforcement

When a metered limit is reached, the **automation engine skips the costly action** (the message or AI call) for that month — it never crashes the flow, and inbound is always processed. Enforcement only applies to automation; set limits to `0` to disable.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `tenant não identificado` | Missing `X-Tenant-ID` | Pass the tenant header |
| `falha ao salvar plano` | DB error on PUT | Retry; check the payload types (limits are integers) |
| Usage all zero | Nothing metered yet this period | Counters start at first automation event of the month |
