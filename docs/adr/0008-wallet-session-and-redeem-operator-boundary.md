# ADR-0008: Wallet sessions and bounded operator redeem submission

**Date**: 2026-08-28  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

Owner-scoped position reads and mutations need authentication without custody. DreamDEX `redeemFor` accepts an owner signature but the bounded operator/worker may submit the transaction after validating that signature. The frontend needs a deterministic contract for both steps.

## Decision

Authenticate owner-scoped API calls with a one-time wallet challenge and expiring opaque bearer session bound to the checksummed address and chain. For redemption, the browser signs the server-created bounded EIP-712 intent with wagmi/viem and submits the signature to the API; the API validates domain/nonce/deadline/owner binding and queues one bounded operator submission. The position projection is the only UI confirmation source.

## Alternatives considered

### Browser submits the `redeemFor` transaction directly

- **Pros**: no operator API for signatures.
- **Cons**: the contract flow is operator-oriented and would require the browser to own submission details that may not be valid for the selected venue/operator.
- **Why not**: keep protocol/operator mechanics behind the adapter and worker boundary.

### Server-held owner keys

- **Pros**: simple background execution.
- **Cons**: custodial risk and incompatible consent model.
- **Why not**: violates ADR-0004.

### Long-lived address-only authorization

- **Pros**: fewer session requests.
- **Cons**: replay and stolen-session exposure.
- **Why not**: challenge, expiry, and address matching provide a bounded owner session.

## Consequences

### Positive

- Clear owner authorization without private-key custody.
- Replay-resistant, intent-bound redeem flow.
- UI can expose `AWAITING_SIGNATURE`, `SUBMISSION_QUEUED`, `SUBMITTED`, `CONFIRMED`, `FAILED`, and `MANUAL_ACTION_REQUIRED` consistently.

### Negative

- Requires auth/session endpoints, secure cookie or bearer handling, and operator queue observability.
- A signed intent can still fail on-chain; manual fallback remains mandatory.

### Risks

- **Signature leakage:** never log raw signatures; redact request bodies and store only hashes/metadata needed for audit.
- **Replay:** persist nonce/intent consumption before queueing and reject expired or already-used intents.
- **Session theft:** use short expiry, secure transport, origin/CSRF controls, and address/resource matching.
