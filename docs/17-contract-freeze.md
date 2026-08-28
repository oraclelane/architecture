# 17. Contract freeze

**Freeze date:** 2026-08-27  
**Contract version:** `api/v1` baseline  
**Authority:** [`docs/contracts/oraclelane.openapi.yaml`](contracts/oraclelane.openapi.yaml)

This is a **design-input freeze**: the stable shapes are sufficient to produce Figma and contract-valid frontend mocks. It is not an integration freeze until the blockers in [frontend implementation spec](23-frontend-implementation-spec.md#contract-blockers-before-implementation) are resolved.

## What is frozen

The following shapes are stable inputs for UI/UX and frontend slicing:

- `Market` and `Outcome` discovery data;
- thesis direction, confidence band, evidence, model version, and expiry;
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
| `thesis-valid` | citations, uncertainty, and future expiry |
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
