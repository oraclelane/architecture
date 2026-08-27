# ADR-0004: Browser wallet signing; no server custody

**Date**: 2026-08-27  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

Users must trust a trading experience with real wallet consequences. Server-held keys would expand security and regulatory scope.

## Decision

The server prepares and validates calldata; the browser wallet displays and signs the transaction. The server stores only public addresses, hashes, receipts, and bounded intents.

## Alternatives considered

### Custodial service signer

- **Pros**: smooth automation.
- **Cons**: key compromise and consent ambiguity.
- **Why not**: incompatible with the MVP trust promise.

### Smart-account abstraction

- **Pros**: future sponsored automation.
- **Cons**: more contracts and audit surface.
- **Why not**: defer until post-hackathon evidence justifies it.

## Consequences

Wallet rejection and pending states are first-class UX states; reactivity uses only bounded owner-authorized redemption.
