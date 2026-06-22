---
name: webhook-setup
description: Subscribe to eduuw events with HMAC-signed webhook deliveries.
---

# Webhook Setup

## When to Use

Receive events (message status, inbound conversations, and more) at your own HTTPS endpoint. Base + auth: see `eduuw-rules`.

## How it works

You register a **subscription**: a target URL + a set of event topics, scoped to your tenant. Each delivery is **signed with HMAC-SHA256** using a secret returned once at creation. Verify the signature on your endpoint.

The webhooks self-service lives on the eduue API at `/api/v1/auth/.../webhooks/self` — exposed to your org with `X-API-Key` + `X-Tenant-ID` (same M2M credential as the rest).

## List contracted topics

```bash
curl https://api.eduue.com.br/api/v1/webhooks/self/topics \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# -> { "topics": ["payments.payment.captured.v1", "education.evasao.risco.v1", ...] }
```
Only contracted topics can be subscribed.

## Create a subscription

```bash
curl -X POST https://api.eduue.com.br/api/v1/webhooks/self \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"target_url":"https://seu-servico.com/webhooks/eduuw","event_topics":["..."]}'
# -> { "subscription": { "id":"...", "target_url":"...", "active":true }, "secret":"whsec_..." }
```
**Save the `secret`** — it is shown only on creation and is used to verify signatures.

## List / deactivate

```bash
curl https://api.eduue.com.br/api/v1/webhooks/self \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl -X DELETE https://api.eduue.com.br/api/v1/webhooks/self/<SUBSCRIPTION_ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```
You can also manage all of this in the dashboard page **Webhooks**.

## Verifying deliveries

Each POST to your `target_url` carries an HMAC-SHA256 signature header computed over the raw body with your `secret`. Recompute and compare in constant time:

```js
import crypto from 'node:crypto'
function verify(rawBody, signatureHeader, secret) {
  const expected = crypto.createHmac('sha256', secret).update(rawBody).digest('hex')
  return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(signatureHeader))
}
```

## Delivery envelope

```json
{ "id": "evt_...", "topic": "education.evasao.risco.v1", "timestamp": "2026-06-22T...Z", "payload": { ... } }
```
Respond `2xx` quickly; failures are recorded in `last_error` and the dashboard.
