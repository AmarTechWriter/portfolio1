# Internal Developer Portal Strategy

The docs above are what an *external* integrating developer sees. This document covers the internal side: how engineering, product, solutions engineering, and support stay aligned on the same API — because a documentation function that only ships external-facing pages will drift from the product within two release cycles.

## Purpose

The internal portal is the **source of truth for what's actually shipping**, ahead of and independent from what's published externally. It exists so that:

- Engineers can see the documented contract for an endpoint *before* they build against it, catching spec/implementation mismatches early.
- Solutions Engineering and Support can look up exact request/response behavior, known edge cases, and internal-only fields without asking an engineer.
- Nothing gets externally published that hasn't first been reviewed against what the API actually does.

## What Lives Here (and Not Externally)

| Content | Why It's Internal-Only |
|---|---|
| Feature flags and rollout status per endpoint | Not relevant or stable enough for external consumers |
| Internal field names and unreleased parameters | Pre-GA, subject to change |
| Acquirer-specific routing logic and known quirks | Competitive/operational detail, not needed for integration |
| Runbooks: "customer says X, here's what's actually happening" | Support/SE tooling, not documentation in the traditional sense |
| Architecture decision records for the API itself | Engineering history, not integration guidance |

## Structure

```
/internal-docs
  /api-contracts/          ← DITA-style reference topics, one per endpoint, pre-release
  /runbooks/                ← support & SE troubleshooting playbooks
  /architecture/             ← ADRs, routing logic, acquirer integration notes
  /release-notes-draft/      ← working notes before they're polished for external release notes
```

## Governance & Workflow

1. **Docs-before-code-freeze**: an endpoint doesn't merge without its internal API contract topic being at least drafted, so documentation is never a post-release afterthought.
2. **Single-source, dual-output**: internal contract topics and external reference pages share the same underlying request/response examples (validated against the OpenAPI spec) so we're never maintaining two versions of the same payload by hand. Internal-only fields and notes are layered on top via metadata flags, not a forked copy.
3. **Review gate**: no external doc ships without a corresponding internal contract review from the owning engineer — this is the mechanism that keeps the external docs accurate, not a separate QA pass bolted on afterward.
4. **Feedback loop from Support/SE**: runbooks get updated the same week a recurring customer issue surfaces a documentation gap — internal docs are living, not archival.

## Tooling

Internal docs live in the same Git repo as the codebase (docs-as-code), reviewed via the same pull-request process as code — an engineer changing an endpoint's behavior updates its contract topic in the same PR. This is deliberately lower-friction and less polished than the external portal; the goal is accuracy and proximity to the code, not presentation.
