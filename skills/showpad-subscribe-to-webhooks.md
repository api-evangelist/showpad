---
name: Subscribe to Showpad webhooks and verify signatures
description: Create a webhook subscription over the Showpad webhooks v1 API, verify the HMAC-SHA256 signature on every delivery, and handle retries correctly.
api: asyncapi/showpad-webhooks.yml
operations:
  - createSubscription
  - subscriptions
  - getSubscriptionById
  - updateSubscription
  - deleteSubscription
  - getSubscriptionLogs
generated: '2026-08-14'
method: generated
source: https://developer.showpad.com/docs/webhooks + asyncapi/showpad-webhooks.yml
---

# Subscribe to Showpad webhooks and verify signatures

Showpad pushes events (shares, views, Shared Spaces, course completions) to an endpoint you
own. There is no AsyncAPI document — the event schemas live in Showpad's Payload Reference.

## Base URL

`https://{subdomain}.api.showpad.com/webhooks/v1`

## Prerequisites

- Plan: Ultimate with Enterprise add-on, Advanced, or Expert.
- Administrator access to the Showpad Admin App.
- An OAuth 2.0 access token or a developer API token.

## Steps

1. **Create the subscription.** `createSubscription` (`POST /subscriptions`) with the endpoint
   URL, a name, the `types[]` you want, an HTTP `method` (`GET` or `POST`), and any custom
   `headers[]`.
2. **Store the signature key immediately.** The response carries `signatureKey`. It is shown
   **once**. If you lose it you cannot verify deliveries and must recreate the subscription.
3. **Verify every delivery** before acting on it:
   - Read `x-showpad-signature-timestamp`. Reject the request if it is older than **5 minutes**.
   - Build `signed_payload` = *raw request body* + `.` + the timestamp string.
     Example: `{ "hello": "world" }.1669302166`
   - Compute HMAC-SHA256 over `signed_payload` with the signature key, Base64-encode it.
   - Split `x-showpad-signature-v1` on commas; accept if **any** entry matches exactly.
   - Use the **raw** body. Any re-serialization breaks verification.
4. **Return the right status.** Showpad reads your response code:
   - `2xx` — delivered, done.
   - `4xx` — treated as failed, and **no retry is attempted**. Never return 4xx for a transient
     problem on your side.
   - `5xx` — retried up to 3 times at 3s, 7.5s and 14.25s. After that the event is lost, so
     persist first and process asynchronously.
5. **Inspect and maintain.** `subscriptions` lists them, `getSubscriptionById` reads one,
   `updateSubscription` (`POST /subscriptions/{id}`) changes it, `deleteSubscription` removes
   it and its log history, `getSubscriptionLogs` (`GET /subscriptions/{id}/logs`) shows
   delivery attempts, response codes and errors.

## Payload shape

```json
{
  "type": "shared-space-user-added",
  "specversion": 1,
  "id": "c40b30f2-fabf-4c89-aed0-f5f5aaac9a24",
  "time": "2022-15-30T06:45:00.250Z",
  "data": { }
}
```

`data` varies by event type. Event names confirmed in Showpad's own docs include
`shared-space-created`, `shared-space-user-added` and `COURSE_COMPLETED`; the complete list is
in the Payload Reference.

## Cautions

- The total retry window is under 25 seconds. Acknowledge fast and process out of band.
- Deleting a subscription permanently removes its log history.
