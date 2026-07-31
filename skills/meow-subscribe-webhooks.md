---
name: Subscribe to and verify webhooks
description: Create a webhook subscription, verify Standard Webhooks HMAC signatures, handle out-of-order deliveries, send a test event, and redrive failures.
api: openapi/meow-openapi.yaml
operations:
  - create_subscription_webhooks_subscriptions_post
  - send_test_event_webhooks_subscriptions__subscription_id__test_post
  - list_deliveries_webhooks_deliveries_get
  - redrive_delivery_webhooks_deliveries__delivery_id__redrive_post
auth: x-api-key header with webhooks:read / webhooks:write
base_url: https://api.meow.com/v1 (sandbox https://api.sandbox.meow.com/v1)
---

# Subscribe to and verify webhooks

Receive events when a transfer changes state or a deposit clears. The wire format
is Standard Webhooks (standardwebhooks.com).

## Rules
- Needs `webhooks:write` to create and `webhooks:read` to inspect.
- The endpoint must be a public HTTPS URL; loopback/metadata IPs are blocked.
- Save the `signing_secret` from the create response — it is returned once.
- Verify every delivery: HMAC-SHA-256 over `"{webhook-id}.{webhook-timestamp}.{body}"`
  using the base64-decoded `whsec_` secret, constant-time compare, and reject
  timestamps older than 5 minutes.
- Dispatch on `data.status`, dedupe on `webhook-id`, and drop stale states using
  the per-resource `sequence` (monotonic, not gapless). See
  asyncapi/meow-webhooks.yml and conventions/meow-conventions.yml.

## Steps
1. **Subscribe** — `create_subscription_webhooks_subscriptions_post`
   (`POST /webhooks/subscriptions`) with `url`, `event_types` (or `null` for all),
   and `payload_mode` (`snapshot` or `thin`).
2. **Send a test event** — `send_test_event_webhooks_subscriptions__subscription_id__test_post`
   (`POST /webhooks/subscriptions/{subscription_id}/test`) to verify your receiver
   and signature handling.
3. **Inspect deliveries** — `list_deliveries_webhooks_deliveries_get`
   (`GET /webhooks/deliveries`) with event_type/resource/sequence metadata.
4. **Redrive a failure** — `redrive_delivery_webhooks_deliveries__delivery_id__redrive_post`
   (`POST /webhooks/deliveries/{delivery_id}/redrive`).
