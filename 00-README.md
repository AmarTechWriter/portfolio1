# Payment Orchestration API — Developer Documentation Suite

A complete, self-contained developer documentation project for a payment orchestration API, built end-to-end: quickstart, authentication, full API reference, webhooks, error handling, and the underlying documentation system design (internal vs. external developer portals, platform structure).

Every example in this project — endpoints, payloads, error codes, webhook signatures — is written to the standard of a real production developer portal. The API itself (**Meridian Pay**) is a representative payment orchestration product, modeled on the scope of an actual implementation. It reflects the scope typical of multi-acquirer routing, alternative payment methods, and transaction lifecycle management.

## What's Included

| File | Contents |
|---|---|
| `docs/01-quickstart.md` | First successful API call, end to end |
| `docs/02-authentication.md` | API key auth + OAuth 2.0 for platform integrations |
| `docs/03-api-reference.md` | Core endpoints: create/capture/refund payments, list transactions, idempotency, rate limits |
| `docs/04-webhooks.md` | Event types, payload structure, signature verification, retry handling |
| `docs/05-error-handling.md` | Error taxonomy, status codes, decline codes, retry guidance |
| `docs/06-internal-developer-portal.md` | How the same API is documented for an internal engineering/support audience |
| `docs/07-external-developer-portal.md` | Information architecture and documentation platform evaluation for the public-facing portal |

## Documentation Approach

- **Docs-as-code**: everything here is Markdown, structured to live in Git and build via CI/CD into a documentation site — no proprietary authoring lock-in. (See [`07-external-developer-portal.md`](docs/07-external-developer-portal.md) for how platform choice adds interactive features like a live API explorer on top of this.)
- **Single-sourcing**: the API reference is written so its request/response examples could be generated or validated directly from an OpenAPI spec rather than hand-maintained separately.
- **Two audiences, one source of truth**: internal (engineering, support) and external (integrating developers) content shares the same underlying request/response data rather than forking into two disconnected sets. This project implements it as separate internal/external doc sets (see [`06-internal-developer-portal.md`](docs/06-internal-developer-portal.md) and [`07-external-developer-portal.md`](docs/07-external-developer-portal.md)) — the same outcome can also be achieved via a single portal with role-based access control gating internal-only content instead.
- **Developer-first writing**: every guide is built around getting a developer to a working integration as fast as possible — real curl commands, real payloads, no filler.

## Skills Demonstrated

API documentation (REST, webhooks, authentication, payment flows) • OpenAPI-aligned reference writing • Information architecture for developer portals • Documentation platform evaluation • Docs-as-code workflows (Git/Markdown/CI-CD) • Internal vs. external content strategy • Error and troubleshooting documentation
