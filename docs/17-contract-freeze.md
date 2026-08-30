# 17. Contract freeze

**Freeze date:** 2026-08-27  
**Contract version:** `api/v1` baseline  
**Authority:** [`docs/contracts/oraclelane.openapi.yaml`](contracts/oraclelane.openapi.yaml)

This is a **design-input freeze** and now includes the resolved owner session, position detail, and signed redeem boundaries. The provider and consumer must still add contract tests for these paths before integration work is considered complete.

**Slice 2 amendment (2026-08-30):** the thesis boundary is accepted and frozen through [ADR-0010](adr/0010-bind-theses-to-market-and-evidence.md). It adds server-owned market/evidence binding, idempotency, drivers/risks, cache metadata, and distinct rejected/unavailable errors. Frontend implementation may begin against generated types after the required contract fixtures are added.

## What is frozen

The following shapes are stable inputs for UI/UX and frontend slicing:

- `Market` and `Outcome` discovery data;
- thesis market/evidence binding, direction, confidence band, drivers, risks, evidence, prompt/model versions, and expiry;
- deterministic `SafetyDecision` and `TradePreview`;
- transaction observation status;
- position lifecycle and settlement fields;
- redeem intent status and typed-data payload envelope;
- pagination metadata and error envelope.

## Required fixtures before frontend slicing

Fixtures must validate against the OpenAPI document and cover:

| Fixture | Required state |
|---|---|
| `market-open-fresh` | tradeable market with two outcomes |
| `market-stale` | rendered but trade blocked |
| `market-locked` | read-only; no preview allowed |
| `thesis-valid-up` | citations, uncertainty, snapshot/evidence binding, and future expiry |
| `thesis-valid-no-trade` | complete successful `NO_TRADE` artifact |
| `thesis-expired` | evidence remains visible but artifact is unusable for preview |
| `thesis-rejected` | invalid model output/citations produce no partial artifact |
| `thesis-unavailable` | API degradation without fabricated advice |
| `preview-allow` | all safety checks pass |
| `preview-blocked` | one or more deterministic checks fail |
| `position-pending` | wallet submitted, receipt pending |
| `position-finalized` | non-void payout vector resolved |
| `position-void` | no winning outcome; manual/refund state |
| `redeem-awaiting-signature` | bounded EIP-712 payload |
| `redeem-manual-fallback` | automation unavailable or expired |

## Change protocol

1. State the consumer need and compatibility impact in a pull request.
2. Change the OpenAPI or event artifact first.
3. Update examples and fixtures in the same change.
4. Regenerate shared types/clients; never maintain handwritten duplicates.
5. Run schema validation, provider serialization tests, and consumer fixture tests.
6. For breaking changes, create a new API version or migration note.

## What UI may assume

- IDs and amounts are opaque decimal strings.
- `freshness` is explicit and can be `FRESH`, `STALE`, or `UNKNOWN`.
- Every action response includes an expiry or lifecycle status where timing matters.
- A `BLOCK` safety decision is final for that preview and cannot be overridden client-side.
- `winningOutcome` is nullable and must remain null for `VOID`.

## What UI must not assume

- That a thesis exists for every market.
- That a quote remains valid after its expiry or a market version change.
- That a submitted transaction is confirmed merely because a hash exists.
- That auto-redeem will always succeed.
- That protocol addresses, decimals, or token symbols are globally static.

## What Slice 2 UI may assume after ADR-0010 is accepted

- A successful thesis is bound to `chainId`, `marketId`, `marketVersion`, and `evidenceHash`.
- `NO_TRADE` is a successful direction with complete drivers, risks, citations, and expiry.
- `502` means the candidate was rejected and no partial model output is safe to render.
- `503` means the thesis path is temporarily unavailable while Case File facts remain usable.
- A thesis at or after `expiresAt` cannot feed a later preview.

## What Slice 2 UI must not assume

- That an old thesis matches a refreshed market snapshot.
- That client-submitted prompts/evidence/model selection are supported.
- That confidence is a probability, guarantee, or deterministic safety decision.
- That retrying a rejected candidate will produce a valid result.
