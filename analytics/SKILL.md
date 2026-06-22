---
name: analytics
description: WhatsApp messaging and conversation analytics — volumes, delivery rates, cost, and CSV export via the eduuw API.
---

# Analytics

## When to Use

Reading messaging metrics for a connected number: sent/delivered volumes over time, conversation/cost analytics (Meta categories), delivery rates, and CSV export. For per-tenant **billing/usage** counters (automation consumption + quotas) use `billing`; for agent-performance CX metrics use `cx-insights`.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. Scope `whatsapp:read`. See `eduuw-rules`.

## Messaging analytics

`start`/`end` are unix seconds; `granularity` = `HALF_HOUR | DAY | MONTH`.

```bash
curl "https://api.eduue.com.br/ext/v1/whatsapp/analytics/messages?start=1718000000&end=1719000000&granularity=DAY" \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "granularity":"DAY","points":[{"start":..,"end":..,"sent":150,"delivered":145}],
#     "totalSent":150,"totalDelivered":145 }
```

## Conversation & cost analytics

`granularity` = `DAILY | MONTHLY`. Returns conversations and cost (USD) by category.

```bash
curl "https://api.eduue.com.br/ext/v1/whatsapp/analytics/conversations?start=1718000000&end=1719000000&granularity=DAILY" \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Delivery rates

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/analytics/delivery-rates \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# → { "deliveryRate": 0.97 }
```

## CSV export

Returns a CSV of messaging analytics for the window (same query params as `messages`):

```bash
curl -X POST "https://api.eduue.com.br/ext/v1/whatsapp/analytics/export?start=1718000000&end=1719000000&granularity=DAY" \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -o analytics.csv
```

## Notes

- Messaging/conversation analytics come from Meta — they need a connected, approved number; new tenants return empty until traffic exists.
- Delivery rate is derived from the local 24h message state, not Meta.

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `account not found for tenant` | No connected number | Connect a number (`onboarding`) |
| empty points | No traffic in the window | Widen the period / send messages first |
