---
name: Create a Showpad Shared Space and add content
description: Stand up a digital sales room from a template, add assets to it, attach quick actions, and hand it to an owner.
api: openapi/showpad-v4-openapi-original.yml
operations:
  - shared-space-templates-list
  - shared-space-templates-get
  - theme-list
  - shared-space-create
  - shared-spaces-items-add
  - shared-spaces-quick-actions-list
  - create-shared-space-quick-action
  - shared-space-get
  - shared-space-transfer-ownership
generated: '2026-08-14'
method: generated
source: openapi/showpad-v4-openapi-original.yml + conventions/showpad-conventions.yml
---

# Create a Showpad Shared Space and add content

A Shared Space is Showpad's digital sales room: a buyer-facing page holding curated assets.

## Base URL

`https://{subdomain}.api.showpad.com/v4`

## Authentication

`Authorization: Bearer <token>`. Creating and modifying Shared Spaces is a write operation —
the OAuth client needs the write scope covering the content profile
(`write_contentprofile_management`).

## Steps

1. **Pick a template.** `shared-space-templates-list` (`GET /shared-space-templates`) lists the
   organization's templates; `shared-space-templates-get`
   (`GET /shared-space-templates/{templateId}`) returns one in full. Templates carry the
   layout and default content, so starting from one is almost always preferable to building
   from scratch.
2. **Pick a theme (optional).** `theme-list` (`GET /themes`) returns the sharing themes
   available; `theme-get` (`GET /themes/{themeId}`) returns one.
3. **Create it.** `shared-space-create` (`POST /shared-spaces`). Capture the returned id — the
   path parameter for every subsequent call is `shareId`.
4. **Add content.** `shared-spaces-items-add` (`POST /shared-spaces/{shareId}/items`). Resolve
   asset ids first with `postAssetsQuery` (see the search-and-fetch skill).
5. **Attach quick actions (optional).** `shared-spaces-quick-actions-list`
   (`GET /shared-spaces/{shareId}/quick-actions`) shows what is already attached;
   `create-shared-space-quick-action` (`POST /shared-spaces/{shareId}/quick-actions`) adds one.
6. **Verify.** `shared-space-get` (`GET /shared-spaces/{shareId}`) — read the space back
   before reporting success.
7. **Hand it over (optional).** `shared-space-transfer-ownership`
   (`POST /shared-spaces/{shareId}/transfer-ownership`) moves ownership to the seller who
   will run the deal.

## Errors

RFC 9457 `application/problem+json`. `409 Conflict` is returned on duplicate-resource
conditions. `403` means the scope or the user's permissions do not cover the action.

## Cautions

- **No idempotency key exists.** If `shared-space-create` times out, do **not** retry blindly
  — call `shared-spaces-list` (`GET /shared-spaces`) and check whether the space was in fact
  created. A blind retry creates a duplicate sales room.
- **Deletion is real.** `shared-space-delete` (`DELETE /shared-spaces/{shareId}`) has no test
  mode behind it.
- Subscribe to the `shared-space-created` and `shared-space-user-added` webhooks
  (`asyncapi/showpad-webhooks.yml`) rather than polling for changes.
