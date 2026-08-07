# Webhooks

Webhooks let your application react to payment events in real time — essential for anything asynchronous: delayed settlement, alternative payment methods with redirect flows, or disputes raised after the fact.

## 1. Register an Endpoint

**Dashboard → Developers → Webhooks → Add Endpoint**, or via the API:

```bash
curl -X POST https://api.meridianpay.com/v1/webhook_endpoints \
  -H "Authorization: Bearer $MERIDIAN_API_KEY" \
  -d '{
    "url": "https://yourapp.com/webhooks/meridian",
    "events": ["payment.succeeded", "payment.failed", "refund.succeeded"]
  }'
```

The response includes a `secret` — store this securely; you'll need it to verify incoming payloads.

## 2. Event Types

| Event | Fired When |
|---|---|
| `payment.succeeded` | Payment captured successfully |
| `payment.failed` | Payment attempt declined or errored |
| `payment.pending` | Payment initiated but awaiting async confirmation (e.g. bank redirect methods) |
| `refund.succeeded` | Refund processed successfully |
| `refund.failed` | Refund could not be processed |
| `dispute.created` | A chargeback/dispute was opened against a payment |

## 3. Payload Structure

```json
{
  "id": "evt_7XqK2mNvLt9R",
  "type": "payment.succeeded",
  "created_at": "2026-08-07T09:15:03Z",
  "data": {
    "id": "pay_3Nq8x2KZvRt9Lm",
    "status": "succeeded",
    "amount": 4999,
    "currency": "GBP",
    "reference": "order_10231"
  }
}
```

## 4. Verify the Signature

Every webhook request includes a `Meridian-Signature` header. **Always verify it** before trusting the payload — this confirms the request genuinely came from Meridian Pay and wasn't spoofed.

```python
import hmac
import hashlib

def verify_signature(payload_body: bytes, signature_header: str, endpoint_secret: str) -> bool:
    expected = hmac.new(
        endpoint_secret.encode(),
        payload_body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature_header)
```

If verification fails, return `400` and discard the payload — do not process it.

## 5. Respond Correctly

- Return a `2xx` status **within 5 seconds** to acknowledge receipt.
- Do your actual processing (updating order state, sending confirmation emails) **after** acknowledging, or asynchronously — don't make Meridian Pay wait on your business logic.
- If your endpoint doesn't respond in time or returns a non-2xx, the event is retried with exponential backoff for up to 24 hours.

## 6. Handle Duplicates

Webhook delivery is **at-least-once**, not exactly-once. Use the event `id` to deduplicate on your end if you're not already idempotent:

```python
if event["id"] in already_processed_ids:
    return  # skip duplicate
```

## Testing Webhooks Locally

Use the Meridian CLI to forward events to your local dev server:

```bash
meridian listen --forward-to localhost:3000/webhooks/meridian
```

This prints a local-only signing secret to use during development, and lets you trigger test events on demand:

```bash
meridian trigger payment.succeeded
```
