---
name: Provision Showpad users, groups and divisions
description: Create and maintain Showpad users, attach them to user groups and divisions, and set division permissions, using API v3 with v4 for reads.
api: openapi/showpad-v3-openapi-original.yml
operations:
  - get_users
  - post_users
  - get_users_id
  - put_users_id
  - delete_users_id
  - get_users_me
  - get_users_id_usergroups
  - link_users_id1_usergroups_id2_link
  - unlink_users_id1_usergroups_id2_unlink
  - get_usergroups
  - post_usergroups
  - get_users_id_divisions
  - get_users_id_divisionpermissions
  - post_users_id_divisionpermissions
  - getUsers
  - getUserById
  - getDivisions
generated: '2026-08-14'
method: generated
source: openapi/showpad-v3-openapi-original.yml + openapi/showpad-v4-openapi-original.yml
---

# Provision Showpad users, groups and divisions

User management lives in **API v3**. v4 exposes reads only (`getUsers`, `getUserById`), so a
provisioning flow is a v3 flow with v4 available for lookups.

## Base URLs

- v3: `https://{subdomain}.showpad.biz/api/v3`
- v4: `https://{subdomain}.api.showpad.com/v4`
- SCIM 2.0: `https://{subdomain}.showpad.biz/api/Users/scim/v2`

## Authentication and scope

`Authorization: Bearer <token>`.

- Reads need `read_user_management`; writes need `write_user_management`.
- Division work also needs `read_division_management` / `write_division_management`.

## Prefer SCIM for bulk provisioning

If the goal is to keep Showpad in sync with an identity provider (Okta, Entra ID, OneLogin),
use Showpad's **SCIM 2.0** endpoints rather than these operations. SCIM is the supported path
for automated provisioning and deprovisioning; the operations below are for targeted work.
See https://developer.showpad.com/docs/apis/users/SCIM.

## Steps

1. **Confirm who you are.** `get_users_me` (`GET /users/me.json`) — verify the token's identity
   and account type before making changes.
2. **Find or create the user.** `get_users` (`GET /users.json`) to search, `post_users`
   (`POST /users.json`) to create. `get_users_id` reads one, `put_users_id` replaces it,
   `delete_users_id` removes it.
3. **Group membership.** `get_usergroups` (`GET /usergroups.json`) lists groups;
   `post_usergroups` creates one. Attach with
   `link_users_id1_usergroups_id2_link` (`POST /users/{id1}/usergroups/{id2}/link.json`) and
   detach with `unlink_users_id1_usergroups_id2_unlink`. Read current membership with
   `get_users_id_usergroups`.
4. **Divisions and permissions.** `get_users_id_divisions` shows the user's divisions;
   `get_users_id_divisionpermissions` and `post_users_id_divisionpermissions` read and set
   division permissions. `getDivisions` (v4) is the cleaner listing.

## Errors

v3 does **not** use RFC 9457. It returns:

```json
{ "response": { "code": 3004, "name": "ResourceNotFoundError", "message": "..." } }
```

Read `response.code` against the registry in `errors/showpad-problem-types.yml`:
`2xxx` parameter, `3xxx` persistence, `4xxx` validation, `5xxx` body. `3004` is a missing
resource; `3002` means the relationship you tried to create already exists — treat that as
success on a link operation, not as a failure.

## Cautions

- **No idempotency key.** `post_users` is not replay-safe. On a timeout, search with
  `get_users` before retrying, or you will create a duplicate user.
- **30 v3 operations are flagged deprecated** in the published spec (mostly asset and division
  paths). None of the user operations above are among them, but check
  `lifecycle/showpad-lifecycle.yml` before adopting other v3 endpoints.
- Paginate with `limit` and `offset`; read `meta.count`.
