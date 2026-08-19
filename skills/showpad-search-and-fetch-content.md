---
name: Search and fetch Showpad sales content
description: Find assets in a Showpad organization with ShowQL and retrieve their files, using the Showpad API v4.
api: openapi/showpad-v4-openapi-original.yml
operations:
  - postAssetsQuery
  - getAssets
  - getAsset
  - getAssetFiles
  - getAssetFileById
generated: '2026-08-14'
method: generated
source: openapi/showpad-v4-openapi-original.yml + conventions/showpad-conventions.yml
---

# Search and fetch Showpad sales content

Showpad's content API is tenant-scoped. Every call goes to the customer's own host.

## Base URL

`https://{subdomain}.api.showpad.com/v4` — `{subdomain}` is the customer's Showpad
subdomain. There is no shared/global host; if you do not know the subdomain, stop and ask.

## Authentication

Send `Authorization: Bearer <token>` on every request. The token is either an OAuth 2.0
access token or a personal API token issued in the Showpad Admin App.

- Access tokens expire after **1 hour**; refresh tokens after **14 days**.
- Reading content requires the `read_contentprofile_management` scope.
- A `401` means the token is missing or expired — refresh, do not retry the same token.
- A `403` means the token is valid but the scope or the user's permissions do not cover the
  resource. Do not retry; report which scope is missing.

## Steps

1. **Search.** Call `postAssetsQuery` (`POST /assets/query`) with a ShowQL query body to find
   assets by name, type, description, content, tags, dates, size, language or engagement.
   Use `getAssets` (`GET /assets`) only when you want an unfiltered listing.
2. **Page.** Use `limit` and `offset`. Read `meta.count` for the total and `response.items`
   for the page. Do not assume a page size — always read `meta.count` before paging further.
3. **Trim the payload.** Pass `fields` with a comma-separated attribute list to keep responses
   small. This matters when handing results to a model.
4. **Read one asset.** Call `getAsset` (`GET /assets/{assetId}`) for the full record.
5. **Get the file.** Call `getAssetFiles` (`GET /assets/{assetId}/files`) to list the asset's
   file representations, then `getAssetFileById` (`GET /assets/{assetId}/files/{fileId}`) for
   a specific one.

## Errors

v4 returns RFC 9457 `application/problem+json` with `title`, `detail` and `status`.

| Status | Meaning | What to do |
|---|---|---|
| 400 | Bad Request | Read `meta[]` in the problem body — it names the offending field, reason and category. |
| 401 | Unauthorized | Refresh the access token. |
| 403 | Forbidden | Missing scope or permission. Report it; do not retry. |
| 404 | Not Found | The asset does not exist **or** the user cannot see it. Both look identical. |
| 429 | Too Many Requests | Back off exponentially. Showpad publishes no numeric limit and no rate-limit headers. |

## Cautions

- **There is no idempotency key.** Showpad publishes no idempotency contract, so never
  blind-retry a write. Retries are safe only on GET.
- **There is no sandbox.** Every call runs against real organization data.
