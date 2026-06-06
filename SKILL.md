---
name: selcom-integration
description: Build, review, or debug Selcom API integrations for Tanzania payments, including checkout create-order-minimal plus wallet-payment USSD push, C2B webhook lookup/validation/notification handling, HMAC signing headers, order-status polling, Laravel/PHP implementations, and common Selcom misconfigurations such as using C2B payload fields on checkout wallet-payment.
---

# Selcom Integration

## Core Workflow

1. Identify the Selcom product path before coding:
   - Checkout USSD push: `POST /checkout/create-order-minimal` then `POST /checkout/wallet-payment`.
   - Checkout status: `GET /checkout/order-status?order_id=...`.
   - C2B collection: direct C2B endpoints and webhook lookup/validation/notification callbacks.
2. Load only the relevant reference:
   - Checkout wallet push: read `references/checkout-wallet-push.md`.
   - C2B webhooks and validation: read `references/c2b-webhooks.md`.
3. Verify credentials are read from config/env only. Never hardcode or commit API keys, API secrets, vendor PINs, webhook tokens, or real merchant identifiers.
4. Preserve exact signing order. Build `Signed-Fields` from the actual payload/query key order and construct the digest string as `timestamp=<Timestamp>&field=value...`.
5. Add HTTP fakes or mocks before tests. Do not let automated tests hit live Selcom.

## Implementation Rules

- Use `Authorization: SELCOM <base64(API_KEY)>`, `Digest-Method: HS256`, `Digest`, `Timestamp`, and `Signed-Fields` on every Selcom request.
- Include `Content-Type: application/json` and `Accept: application/json`.
- Format timestamps as ISO 8601 with timezone offset, for example `Y-m-d\TH:i:sP`.
- Use integer TZS amounts when Selcom expects checkout order amounts.
- Normalize Tanzanian MSISDNs to `255XXXXXXXXX` before sending.
- Treat a successful push response as "prompt sent", not "money collected".
- Query status or process webhooks to mark payment completion.

## Common Fixes

- For checkout wallet push, do not send `vendor`, `pin`, `utilityref`, or `amount` to `/checkout/wallet-payment`.
- For checkout wallet push, do send both `transid` and `order_id`, with both values equal to the original order id.
- For checkout order status, use `GET`, not `POST`.
- For GET requests, sign the same query parameters that are sent in the URL.
- If Selcom returns missing-parameter errors on wallet payment, first check that `order_id` exists alongside `transid`.

## Laravel/PHP Guidance

- Keep Selcom signing and HTTP calls in a service class.
- Keep payment lifecycle orchestration in a separate payment service.
- Log outgoing request payloads and incoming responses without secrets.
- Use queues for merchant callback forwarding.
- Use `Http::fake()` in Laravel tests and assert request body, URL, and `Signed-Fields`.

