# Error Handling

Meridian Pay uses conventional HTTP status codes and a consistent error body so failures are predictable to handle programmatically.

## Error Response Shape

```json
{
  "error": {
    "type": "card_error",
    "code": "card_declined",
    "message": "The card was declined by the issuing bank.",
    "decline_code": "insufficient_funds",
    "request_id": "req_9KpL2mNvXt7Q"
  }
}
```

Always log `request_id` — it's what support needs to trace a specific call server-side.

## HTTP Status Codes

| Status | Meaning |
|---|---|
| `400` | Malformed request — check parameter types and required fields |
| `401` | Authentication failed — see [Authentication Errors](#authentication-errors) |
| `403` | Authenticated, but not permitted to perform this action |
| `404` | Resource doesn't exist (wrong ID, or wrong environment — test ID used against live) |
| `409` | Conflict — e.g. attempting to capture an already-captured payment |
| `410` | Resource expired — e.g. uncaptured authorization past the 7-day window |
| `422` | Request understood but semantically invalid (e.g. refund amount exceeds original payment) |
| `429` | Rate limit exceeded |
| `500` / `503` | Meridian Pay-side error — safe to retry with backoff |

## Error Types

| `type` | Meaning | Typical Handling |
|---|---|---|
| `card_error` | Card declined, expired, or invalid | Show the customer a friendly message; do not retry automatically |
| `validation_error` | Request payload failed validation | Fix the request — this is a bug in your integration, not a payment failure |
| `authentication_error` | Bad or missing API key/token | Check key validity and environment |
| `idempotency_error` | Same `Idempotency-Key` reused with a different request body | Use a unique key per distinct request |
| `api_error` | Something went wrong on Meridian Pay's side | Safe to retry with exponential backoff |

## Authentication Errors

| `code` | Cause | Fix |
|---|---|---|
| `invalid_api_key` | Key is malformed or has been revoked | Check the key in Dashboard → API Keys |
| `environment_mismatch` | Test key used against a live-only resource, or vice versa | Confirm you're using the right key for the environment |
| `insufficient_scope` | OAuth token missing required scope | Re-authorize with the correct `scope` parameter |

## Common Decline Codes

| `decline_code` | Meaning | Suggested Customer Message |
|---|---|---|
| `insufficient_funds` | Card has insufficient balance | "Your card was declined due to insufficient funds." |
| `expired_card` | Card expiry date has passed | "Your card has expired — please use a different card." |
| `fraud_suspected` | Issuer flagged as potentially fraudulent | "Your card was declined. Please contact your bank or try another payment method." |
| `do_not_honor` | Generic issuer decline, no further detail given | "Your card was declined — please try another payment method." |

> Decline codes come from the acquirer/issuer and are intentionally generic for `do_not_honor` — Meridian Pay cannot provide more detail than the issuer gives us.

## Retry Guidance

- **`5xx` and `429`**: safe to retry with exponential backoff (start at 1s, cap at 30s, max 5 attempts).
- **`4xx` (except `429`)**: do not retry as-is — the request itself needs to change (fix the payload, refresh the token, use a valid ID).
- **Card declines**: never auto-retry the same card silently — prompt the customer for a different payment method or updated card details.

## Troubleshooting Checklist

1. Check `request_id` against **Dashboard → Logs** for the full request/response trail.
2. Confirm you're using the correct key for the environment (`sk_test_` vs `sk_live_`).
3. For `422` errors, re-read the `message` field — it names the exact field that failed validation.
4. For intermittent `5xx`s, check [status.meridianpay.com](#) before assuming it's your integration.
