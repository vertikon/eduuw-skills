---
name: broadcast-campaign
description: Send template broadcasts to many recipients (opt-out aware) in eduuw.
---

# Broadcast Campaign

## When to Use

Sending a template message to many recipients at once (a campaign). Base + auth: see `eduuw-rules`.

## Rules

- Broadcasts use an **APPROVED template** (free-form text can't be bulk-sent outside the 24h window). See `whatsapp-templates`.
- **Opt-out is enforced:** recipients with `OPTED_OUT` status are skipped. Manage consent via `optins` (see `contacts-management`).
- Recipients are phone numbers (digits, country code first).

## Create a broadcast

```bash
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/broadcasts \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{
    "name": "Lembrete de matrícula",
    "template_name": "lembrete_matricula",
    "language": "pt_BR",
    "recipients": ["5585992009999","5511988887777"],
    "scheduled_at": "2026-06-23T12:00:00Z"
  }'
```
Required: `template_name`, `recipients`, `scheduled_at` (RFC3339; use now/near-now to send immediately).

## List / cancel

```bash
curl https://api.eduue.com.br/ext/v1/whatsapp/broadcasts \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/broadcasts/<BROADCAST_ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Building the recipient list from a segment

Pull contacts by tag and filter out opt-outs before broadcasting:

```bash
# 1) contacts (with tags) and opt-out status
curl "https://api.eduue.com.br/ext/v1/whatsapp/contacts" -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl "https://api.eduue.com.br/ext/v1/whatsapp/optins?status=OPTED_OUT&limit=5000" -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
# 2) recipients = contacts where tag matches AND phone not in opt-out set
```
See `contacts-management` for the contact/segment model.

## Tips

- Throughput is limited by your number's Meta tier — large lists are dispatched asynchronously.
- Track delivery per recipient via message status (`send-message` → status) or analytics.
