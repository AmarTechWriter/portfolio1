# Authentication

Meridian Pay supports two authentication methods depending on how you're integrating.

| Method | Use When |
|---|---|
| **API Key** | You're integrating your own backend directly (most merchants) |
| **OAuth 2.0** | You're a platform or PSP integrating *on behalf of* multiple merchant accounts |

---

## API Key Authentication

Every request must include your secret key in the `Authorization` header as a Bearer token:

```bash
curl https://api.meridianpay.com/v1/payments \
  -H "Authorization: Bearer sk_test_51Hxx..."
```

### Key Types

| Prefix | Environment | Where It Works |
|---|---|---|
| `sk_test_` | Sandbox | Test acquirers only; no real funds move |
| `sk_live_` | Production | Real acquirers; requires completed merchant verification |
| `pk_...` | Public | Safe to expose client-side (used for tokenizing card data only — never for API calls) |

### Key Rotation

Rotate keys without downtime:

1. Generate a new key in **Dashboard → Developers → API Keys**.
2. Deploy your application with both keys accepted (roll over gradually if you're load-balanced).
3. Revoke the old key once you've confirmed zero requests are using it (check **Dashboard → Developers → Key Usage**).

> **Security:** Never commit API keys to source control. Never log the full key value — Meridian Pay's logs and dashboard only ever display the last 4 characters.

---

## OAuth 2.0 (Platform Integrations)

If you're building a platform where merchants connect their own Meridian Pay account (e.g. a marketplace or a POS provider), use the OAuth authorization code flow instead of sharing API keys.

### Flow Overview

1. **Redirect the merchant** to Meridian Pay's authorization URL:

```
https://connect.meridianpay.com/oauth/authorize
  ?client_id=YOUR_CLIENT_ID
  &redirect_uri=https://yourapp.com/oauth/callback
  &scope=payments:read payments:write
  &response_type=code
```

2. **Merchant approves access** in the Meridian Pay-hosted consent screen.

3. **Exchange the returned code** for an access token:

```bash
curl -X POST https://connect.meridianpay.com/oauth/token \
  -d grant_type=authorization_code \
  -d code=AUTH_CODE_FROM_REDIRECT \
  -d client_id=YOUR_CLIENT_ID \
  -d client_secret=YOUR_CLIENT_SECRET
```

Response:

```json
{
  "access_token": "at_9fKL...",
  "refresh_token": "rt_2xQm...",
  "expires_in": 3600,
  "merchant_id": "mrc_7HqL2Kv"
}
```

4. **Use the access token** as the Bearer token on subsequent calls, scoped to that merchant's account. Refresh it using `refresh_token` before it expires — access tokens are short-lived by design.

### Scopes

| Scope | Grants |
|---|---|
| `payments:read` | View payments and transaction history |
| `payments:write` | Create payments, captures, refunds |
| `webhooks:manage` | Register and manage webhook endpoints |

---

## Common Authentication Errors

See [Error Handling](05-error-handling.md#authentication-errors) for the full list, but the two you'll hit most often during setup:

- `401 invalid_api_key` — key is malformed, revoked, or environment-mismatched (test key against a request that requires live, or vice versa)
- `403 insufficient_scope` — OAuth token doesn't include the scope required for that endpoint
