# External Developer Portal Strategy

This covers the public-facing side of the documentation: information architecture, and how I'd evaluate and select a documentation platform to host it.

## Information Architecture

The docs in this project (`01`–`05`) map onto a standard IA, ordered by developer intent rather than internal team structure:

```
Get Started
  ├─ Quickstart               ← 01-quickstart.md
  ├─ Authentication            ← 02-authentication.md
  └─ Sandbox & Test Cards

Guides
  ├─ Handling Webhooks          ← 04-webhooks.md
  ├─ Delayed Capture Flows
  ├─ Recurring Payments
  └─ Going Live Checklist

API Reference
  ├─ Payments                    ← 03-api-reference.md
  ├─ Refunds
  ├─ Transactions
  └─ Webhook Endpoints

Resources
  ├─ Error Reference             ← 05-error-handling.md
  ├─ Changelog / Release Notes
  ├─ SDKs & Client Libraries
  └─ Status Page
```

**Design principle:** a developer arriving cold should reach a successful test payment without reading anything outside "Get Started." Everything else is there when they need to go deeper — not required reading upfront.

## Platform Evaluation

Comparing common documentation platforms for a fintech/payments developer audience:

| Platform | Strengths | Watch-Outs | Best Fit When |
|---|---|---|---|
| **Mintlify** | Fast to stand up, strong out-of-box design, good OpenAPI auto-reference generation | Less mature for highly custom internal/external content splitting | Reference is OpenAPI-driven and speed-to-launch matters most |
| **ReadMe** | Built-in API explorer (try-it-now), strong analytics on doc usage | Pricier at scale, more SaaS lock-in than pure docs-as-code | Interactive "try this endpoint" is a priority for reducing support load |
| **Fern** | Excellent for generating SDKs + docs from one spec, strong versioning | Smaller ecosystem, newer platform | SDK generation is on the roadmap alongside docs |
| **Docusaurus** | Fully open-source, complete control, no vendor lock-in, plugin ecosystem | More engineering lift to build API-explorer-style features yourself | Long-term platform independence outweighs speed-to-launch |
| **Stoplight** | Strong OpenAPI-first design workflow (design → mock → docs) | More "API design tool that also does docs" than docs-first | API design and documentation process should be unified from day one |

**Recommendation approach:** the right platform depends on two things — how much interactivity (live API explorer, generated SDKs) the developer experience needs, and how much engineering bandwidth is available to support a platform versus needing something turnkey. For a payments API, where "try it before you build against it" meaningfully reduces support load, **ReadMe or Stoplight** are the strongest starting points — validated by piloting real content (like this project) in both before committing.

## Success Metrics

- **Time-to-first-successful-call** — from signup to first `200`/`201` response, measurable via sandbox key creation → first API call timestamp.
- **Support tickets tagged "documentation gap"** trending down over time.
- **Doc search queries with zero results** — a leading indicator of missing content.
