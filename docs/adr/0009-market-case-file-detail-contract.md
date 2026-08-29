# ADR-0009: Use a dedicated detail contract for Market Case File data

**Date**: 2026-08-30  
**Status**: proposed  
**Deciders**: Oraclelane team

## Context

Slice 0 proves market discovery with the list-oriented `Market` schema. Slice 1 needs a decision-quality Case File, but the current `MarketResponse` does not carry resolution terms, lock timing, evidence provenance, or a canonical contract reference.

The frontend must not invent those fields or infer them from `closeAt`. The list response should also remain small and stable for Radar performance.

## Decision

Additive detail data should be represented by a dedicated `MarketCaseFile` response for `GET /markets/{marketId}`. Keep the existing `Market` schema for `/markets` and compose the detail response around it:

```text
MarketCaseFileResponse
└── data: MarketCaseFile
    ├── (all existing Market fields)
    ├── resolution: ResolutionTermsEnvelope
    ├── evidence: EvidenceEnvelope
    └── contract: ContractReference
```

The proposed envelopes use explicit availability rather than silent omission:

- `ResolutionTermsEnvelope.availability`: `AVAILABLE` or `UNAVAILABLE`;
- `EvidenceEnvelope.availability`: `AVAILABLE` or `UNAVAILABLE`;
- `ContractReference.availability`: `AVAILABLE` or `UNAVAILABLE`.

When unavailable, the response includes a safe, user-facing reason and no fabricated fallback. `lockAt` is nullable only when the provider explicitly confirms that the value is not applicable; it must not be derived from `closeAt`.

`MarketCaseFile` is an additive composition of the existing `Market` fields, so existing consumers can continue reading `data.marketId`, `data.question`, `data.status`, and `data.outcomes` without a nesting migration. The detail response preserves the existing `{ data: ... }` envelope and the required `chainId` query parameter. This is a proposed additive contract; the OpenAPI file remains unchanged until DreamDEX field ownership and serialization are confirmed.

In OpenAPI terms, `MarketCaseFile` should use an `allOf` relationship with `Market`, followed by the required detail properties. The list endpoint continues to return `MarketListResponse` with plain `Market` items.

## Proposed fields

| Object | Field | Type | Requirement |
|---|---|---|---|
| `ResolutionTermsEnvelope` | `availability` | enum | Always required. |
| `ResolutionTermsEnvelope` | `sourceName` | string/null | Required when available. |
| `ResolutionTermsEnvelope` | `sourceUrl` | URI/null | Required when available and safe to expose. |
| `ResolutionTermsEnvelope` | `rule` | string/null | Verbatim or provider-approved plain language. |
| `ResolutionTermsEnvelope` | `lockAt` | date-time/null | Provider value only; never inferred. |
| `ResolutionTermsEnvelope` | `voidBehavior` | string/null | Required when available. |
| `ResolutionTermsEnvelope` | `unavailableReason` | string/null | Required when unavailable. |
| `EvidenceEnvelope` | `availability` | enum | Always required. |
| `EvidenceEnvelope` | `items` | array | Empty when unavailable. |
| `EvidenceEnvelope` | `observedAt` | date-time | Required when available. |
| `EvidenceEnvelope` | `unavailableReason` | string/null | Required when unavailable. |
| `ContractReference` | `availability` | enum | Always required. |
| `ContractReference` | `address` | string/null | Chain-scoped address when available. |
| `ContractReference` | `url` | URI/null | Canonical explorer or protocol link when available. |
| `ContractReference` | `unavailableReason` | string/null | Required when unavailable. |

Existing `Market` fields remain the source for identity, question, status, outcomes, close time, observed time, and market freshness. Existing `Evidence` items remain the source for `sourceId`, title, URL, publication time, and relevance unless DreamDEX requires an adapter-specific extension.

## Alternatives considered

### Add all detail fields directly to the shared `Market` schema

- **Pros**: fewer schema names and a simpler generated type.
- **Cons**: list responses become heavier; detail-only provenance is mixed with discovery data; availability semantics become ambiguous.
- **Why not**: preserve a lean Radar payload and a clear detail boundary.

### Make terms and evidence optional properties

- **Pros**: minimal OpenAPI change.
- **Cons**: omission is indistinguishable from provider failure, and clients will be tempted to fabricate or infer values.
- **Why not**: explicit availability is required for a trust-oriented product.

### Derive `lockAt` from `closeAt`

- **Pros**: no new provider field.
- **Cons**: protocol semantics may differ; an incorrect lock time can cause financial harm.
- **Why not**: only provider- or contract-sourced timing is acceptable.

## Consequences

### Positive

- Radar and Case File payloads have separate performance and trust responsibilities.
- Missing provider data is visible and testable.
- Frontend, backend, and DreamDEX adapter can agree on one typed detail boundary before implementation.
- The design supports a read-only Case File without wallet or execution dependencies.

### Negative

- Requires an additive OpenAPI change, regenerated types, provider serialization, and new fixtures.
- The backend must normalize source URLs and availability reasons safely.
- Two response shapes must be documented and maintained.

## Implementation gate

Before changing `oraclelane.openapi.yaml`:

1. Confirm which terms, lock timing, evidence, and contract-reference fields DreamDEX exposes.
2. Confirm whether evidence is protocol-provided or an Oraclelane adapter responsibility.
3. Validate URL safety and source freshness semantics.
4. Add `market-case-file-*` fixtures for available and unavailable variants.
5. Update the OpenAPI document first, regenerate frontend types, then implement provider and UI changes.

Until these gates pass, the Case File design must render typed unavailable states and keep the next-step control inactive.
