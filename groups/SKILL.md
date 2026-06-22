---
name: groups
description: Manage WhatsApp groups — create, update, members, invite links and join requests via the eduuw API.
---

# Groups

## When to Use

Managing WhatsApp groups for the connected number: create/update/delete groups, manage members and roles, invite links, and join requests.

All requests: base `https://api.eduue.com.br/ext/v1/whatsapp`, headers `X-API-Key` + `X-Tenant-ID`. See `eduuw-rules`.

## Groups

```bash
# list / create
curl https://api.eduue.com.br/ext/v1/whatsapp/groups -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/groups \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"name":"Turma Enfermagem 2026","description":"Avisos da turma"}'

# get / update / delete
curl https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID> -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X PUT https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID> \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"name":"Turma Enfermagem 2026/2","description":"..."}'
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID> -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Members & roles

```bash
# list / add / remove members
curl https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/members -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/members \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"phone":"5585992009999"}'
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/members/5585992009999 -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# update role (admin/member)
curl -X PUT https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/members/5585992009999/role \
  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID" -H "Content-Type: application/json" \
  -d '{"role":"admin"}'
```

## Invite links & join requests

```bash
# invite link: get / create / revoke
curl https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/invite-link -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X POST   https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/invite-link -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X DELETE https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/invite-link -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"

# join requests: list / approve / reject
curl https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/join-requests -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/join-requests/<REQ_ID>/approve -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
curl -X POST https://api.eduue.com.br/ext/v1/whatsapp/groups/<GROUP_ID>/join-requests/<REQ_ID>/reject  -H "X-API-Key: $EDUUE_API_KEY" -H "X-Tenant-ID: $TENANT_ID"
```

## Common errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `424 no_whatsapp_number_connected` | No connected number | Connect a number (`onboarding`) |
| 403 | Key missing `whatsapp:write` | Mint a key with write scope |
| member add fails | Number not on WhatsApp / privacy | Verify the phone is a valid WhatsApp user |
