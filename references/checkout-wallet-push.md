# Checkout Wallet Push Reference

Use this reference when implementing Selcom checkout USSD push, especially when migrating from a direct C2B/pushussd implementation.

## Flow

1. Generate a unique order id, for example `KML-YYYYMMDD-HHMMSS-RND`.
2. Create a checkout order:

```http
POST /checkout/create-order-minimal
```

```json
{
  "vendor": "YOUR_VENDOR_ID",
  "order_id": "KML-20260606-103046-583",
  "buyer_email": "customer@example.com",
  "buyer_name": "Customer Name",
  "buyer_phone": "255762817032",
  "amount": 15000,
  "currency": "TZS",
  "no_of_items": 1
}
```

3. If order creation succeeds, trigger wallet payment:

```http
POST /checkout/wallet-payment
```

```json
{
  "transid": "KML-20260606-103046-583",
  "order_id": "KML-20260606-103046-583",
  "msisdn": "255762817032"
}
```

`transid` and `order_id` must both be present and equal to the original order id.

4. Poll status if needed:

```http
GET /checkout/order-status?order_id=KML-20260606-103046-583
```

## Signing

For POST requests, sign the fields in the exact order of the JSON payload.

Example for create order:

```text
Signed-Fields: vendor,order_id,buyer_email,buyer_name,buyer_phone,amount,currency,no_of_items
signing string: timestamp=<Timestamp>&vendor=<Vendor>&order_id=<OrderId>&buyer_email=<Email>&buyer_name=<Name>&buyer_phone=<Phone>&amount=<Amount>&currency=TZS&no_of_items=1
```

Example for wallet payment:

```text
Signed-Fields: transid,order_id,msisdn
signing string: timestamp=<Timestamp>&transid=<OrderId>&order_id=<OrderId>&msisdn=<Phone>
```

Digest:

```php
base64_encode(hash_hmac('sha256', $signingString, $apiSecret, true));
```

## Result Handling

Accept HTTP 200 or 201 with `result/status` equal to `SUCCESS` or `resultcode` equal to `000` as a successful API step.

The wallet-payment success means the USSD prompt was sent. Mark local payment status as `push_sent` or `pending`, not `success`, until webhook/status confirms payment.

## Pitfalls

- Do not call `/checkout/wallet-payment` before `/checkout/create-order-minimal`.
- Do not omit `order_id` from wallet payment.
- Do not use `/wallet/pushussd` payload fields (`utilityref`, `pin`) on checkout wallet payment.
- Do not send decimal strings for checkout amount when an integer is expected.
- Do not use POST for `/checkout/order-status`.

