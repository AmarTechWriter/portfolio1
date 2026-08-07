# API Reference

Base URL: `https://api.meridianpay.com/v1`

All requests and responses use `application/json`. All amounts are integers in the smallest unit of the currency (see [Currencies](#currencies)).

---

## Payment Object

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique payment identifier, prefixed `pay_` |
| `status` | string | `requires_capture`, `succeeded`, `pending`, `failed`, `refunded`, `partially_refunded` |
| `amount` | integer | Amount in smallest currency unit |
| `currency` | string | ISO 4217 currency code |
| `payment_method` | string | Tokenized payment method used |
| `acquirer` | string | Acquirer the transaction was routed to |
| `reference` | string | Your own order/reference ID (optional, echoed back) |
| `created_at` | string | ISO 8601 timestamp |

---

## Create a Payment

`POST /v1/payments`

| Parameter | Type | Required | Description |
|---|---|---|---|
| `amount` | integer | Yes | Amount in smallest currency unit |
| `currency` | string | Yes | ISO 4217 code, e.g. `GBP`, `EUR`, `USD` |
| `payment_method` | string | Yes | Token from your client-side tokenization step |
| `capture` | boolean | No | Default `true`. Set `false` to authorize only (see [Capture a Payment](#capture-a-payment)) |
| `reference` | string | No | Your internal order reference |
| `metadata` | object | No | Arbitrary key-value pairs, returned unchanged |

**Example request**

```bash
curl -X POST https://api.meridianpay.com/v1/payments \
  -H "Authorization: Bearer $MERIDIAN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 12000,
    "currency": "EUR",
    "payment_method": "pm_card_visa_test",
    "capture": false,
    "reference": "order_88213"
  }'
```

**Example response — `201 Created`**

```json
{
  "id": "pay_4Nq1x8KZvRt2Lm",
  "status": "requires_capture",
  "amount": 12000,
  "currency": "EUR",
  "payment_method": "pm_card_visa_test",
  "acquirer": "sandbox-acquirer-01",
  "reference": "order_88213",
  "created_at": "2026-08-07T10:02:11Z"
}
```

---

## Capture a Payment

`POST /v1/payments/{id}/capture`

Captures a previously authorized (uncaptured) payment. Use this for delayed-capture flows — e.g. authorizing at checkout, capturing at shipment.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `amount` | integer | No | Partial capture amount. Omit to capture the full authorized amount. |

```bash
curl -X POST https://api.meridianpay.com/v1/payments/pay_4Nq1x8KZvRt2Lm/capture \
  -H "Authorization: Bearer $MERIDIAN_API_KEY"
```

Returns the updated payment object with `status: "succeeded"`.

> Authorizations expire after **7 days** if uncaptured. Attempting to capture an expired authorization returns `410 authorization_expired`.

---

## Refund a Payment

`POST /v1/payments/{id}/refunds`

| Parameter | Type | Required | Description |
|---|---|---|---|
| `amount` | integer | No | Partial refund amount. Omit for a full refund. |
| `reason` | string | No | `requested_by_customer`, `duplicate`, `fraudulent` |

```bash
curl -X POST https://api.meridianpay.com/v1/payments/pay_4Nq1x8KZvRt2Lm/refunds \
  -H "Authorization: Bearer $MERIDIAN_API_KEY" \
  -d '{"amount": 5000, "reason": "requested_by_customer"}'
```

**Response — `201 Created`**

```json
{
  "id": "rfd_8Kp3mNq7XvLt",
  "payment_id": "pay_4Nq1x8KZvRt2Lm",
  "amount": 5000,
  "status": "succeeded",
  "reason": "requested_by_customer",
  "created_at": "2026-08-07T11:40:00Z"
}
```

Multiple partial refunds are allowed up to the original payment amount. The parent payment's `status` becomes `partially_refunded` or `refunded` accordingly.

---

## List Transactions

`GET /v1/transactions`

| Query Parameter | Type | Description |
|---|---|---|
| `status` | string | Filter by status |
| `created_after` | string | ISO 8601 timestamp |
| `created_before` | string | ISO 8601 timestamp |
| `limit` | integer | Default 25, max 100 |
| `starting_after` | string | Cursor for pagination — pass the last `id` from the previous page |

```bash
curl "https://api.meridianpay.com/v1/transactions?status=succeeded&limit=50" \
  -H "Authorization: Bearer $MERIDIAN_API_KEY"
```

**Response**

```json
{
  "data": [
    { "id": "pay_4Nq1x8KZvRt2Lm", "status": "succeeded", "amount": 12000, "currency": "EUR" }
  ],
  "has_more": true,
  "next_cursor": "pay_4Nq1x8KZvRt2Lm"
}
```

---

## Currencies

Most currencies use a **2-decimal minor unit** (e.g. `4999` GBP = £49.99). Zero-decimal currencies (e.g. `JPY`) are passed as whole numbers with no adjustment — `5000` JPY = ¥5000, not ¥50.00. Full list in the [Currency Support appendix](#).

## Idempotency

All `POST` requests accept an `Idempotency-Key` header. Retrying a request with the same key returns the original result instead of creating a duplicate payment — required for safe network-failure retries.

```bash
curl -X POST https://api.meridianpay.com/v1/payments \
  -H "Idempotency-Key: order_10231-attempt-1" \
  ...
```

## Rate Limits

| Tier | Limit |
|---|---|
| Sandbox | 25 requests/second |
| Live | 100 requests/second (contact us to raise this) |

Exceeding the limit returns `429 rate_limit_exceeded` with a `Retry-After` header.
