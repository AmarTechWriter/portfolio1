# Quickstart: Your First Payment

Get from zero to a successful test payment in under 10 minutes.

## What You'll Need

- A Meridian Pay sandbox account ([sign up free](#))
- Your sandbox API key (found in **Dashboard → Developers → API Keys**)
- `curl`, or any HTTP client

## 1. Get Your API Key

Every request to the Meridian Pay API is authenticated with a key passed in the `Authorization` header.

```bash
export MERIDIAN_API_KEY="sk_test_51Hxx...redacted"
```

> **Sandbox vs. Live:** Sandbox keys are prefixed `sk_test_`, live keys `sk_live_`. Sandbox transactions never touch a real acquirer — see [Authentication](02-authentication.md) for how to promote to live.

## 2. Create a Payment

Send a `POST` request to `/v1/payments` with an amount, currency, and a test payment method token.

```bash
curl -X POST https://api.meridianpay.com/v1/payments \
  -H "Authorization: Bearer $MERIDIAN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 4999,
    "currency": "GBP",
    "payment_method": "pm_card_visa_test",
    "capture": true,
    "reference": "order_10231"
  }'
```

**Amounts are in the smallest currency unit** (e.g. `4999` = £49.99). See [Working with Currencies](03-api-reference.md#currencies) for zero-decimal currency handling.

## 3. Read the Response

A successful request returns `201 Created` with the payment object:

```json
{
  "id": "pay_3Nq8x2KZvRt9Lm",
  "status": "succeeded",
  "amount": 4999,
  "currency": "GBP",
  "reference": "order_10231",
  "acquirer": "sandbox-acquirer-01",
  "created_at": "2026-08-07T09:14:22Z"
}
```

`status` is the field you'll check to confirm the payment went through. See the full list of possible values in the [Payments reference](03-api-reference.md#payment-object).

## 4. Handle the Result in Your Application

At minimum, your integration should:

1. Store the returned `id` against your internal order record.
2. Check `status` — `succeeded` means funds are captured; `pending` means it's awaiting an async confirmation (see [Webhooks](04-webhooks.md) for how to get notified when it resolves).
3. Show the customer a confirmation or failure state based on `status`.

## 5. Go Live

When you're ready to accept real payments:

1. Complete merchant verification in the Dashboard.
2. Swap your sandbox key for a live key (`sk_live_...`).
3. Re-point webhook endpoints to your production URL.
4. Run one real low-value transaction end-to-end before launch.

## Next Steps

- [Authentication](02-authentication.md) — API keys, OAuth for platform integrations, and key rotation
- [API Reference](03-api-reference.md) — full endpoint list: captures, refunds, voids, transaction history
- [Webhooks](04-webhooks.md) — get notified on payment status changes in real time
- [Error Handling](05-error-handling.md) — what to do when a payment fails
