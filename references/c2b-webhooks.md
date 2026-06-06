# C2B Webhooks Reference

Use this reference for Selcom C2B lookup, validation, notification, and direct C2B collection integrations.

## Inbound Authentication

Selcom webhooks commonly use:

```http
Authorization: Bearer <SELCOM_C2B_TOKEN>
Content-Type: application/json
```

Verify the bearer token with constant-time comparison against an environment/config value.

## Webhook Payload Shape

```json
{
  "operator": "AIRTELMONEY",
  "transid": "XYZ123444",
  "reference": "033XX12211",
  "utilityref": "AB12345",
  "amount": 1000,
  "msisdn": "06534567891"
}
```

Lookup local payment by `transid`, then by public payment id or merchant order id when `utilityref` is provided.

## Response Shape

```json
{
  "reference": "033XX12211",
  "resultcode": "000",
  "result": "SUCCESS",
  "message": "Accepted",
  "name": "Customer Name",
  "amount": 1000
}
```

Common result codes:

- `000`: accepted.
- `010`: invalid utility reference.
- `012`: invalid amount.
- `014`: amount too high.
- `015`: amount too low.

## Validation Rules

- Lookup: return success only when the payment/utility reference exists.
- Validation: compare received amount with local expected amount. Failure can trigger reversal, so fail only for true mismatches.
- Notification: update local payment status idempotently and store full payload.
- Duplicate notifications should return success if already processed.
- Forward merchant callbacks asynchronously and never reverse funds because a merchant callback failed.

## Status Mapping

Use project-specific mapping, but a safe default is:

- `000`: success.
- temporary/in-progress codes: in progress.
- `999`: ambiguous.
- other non-success codes: failed.

