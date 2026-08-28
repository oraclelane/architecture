# ADR-0007: RainbowKit, wagmi, and viem wallet stack

**Date**: 2026-08-28  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

Oraclelane needs a familiar wallet onboarding experience, typed React wallet state, and precise EVM transaction/EIP-712 primitives. The browser must remain non-custodial while the application supports Somnia network checks and DreamDEX calldata.

## Decision

Use RainbowKit for wallet connection UX, wagmi for React hooks and wallet/transaction lifecycle, and viem for chain clients, ABI encoding, typed-data signing, and receipt reads. The stack is confined to the browser wallet/protocol boundary; domain policies and API contracts remain framework-independent.

## Alternatives considered

### ethers.js-only integration

- **Pros**: mature low-level API and broad examples.
- **Cons**: more hand-written React lifecycle glue and a second state/query story.
- **Why not**: wagmi + viem gives a cohesive typed React/EVM boundary for this MVP.

### Custom wallet modal and raw RPC

- **Pros**: maximum visual/control flexibility.
- **Cons**: connector, chain-switch, and signing edge cases become team-owned.
- **Why not**: RainbowKit reduces onboarding risk and lets the team focus on Event Contract UX.

### Server-side signer

- **Pros**: simpler automation UX.
- **Cons**: custody, key compromise, and consent risk.
- **Why not**: contradicts ADR-0004 and the non-custodial product promise.

## Consequences

### Positive

- Familiar wallet connection and network switching flow.
- Typed hooks and viem precision for amounts, calldata, EIP-712, and receipts.
- Clear separation between UI, wallet lifecycle, and protocol encoding.

### Negative

- RainbowKit/wagmi/viem versions must remain compatible and be upgraded together.
- Browser-only wallet logic increases the client-component boundary.

### Risks

- **Version drift:** pin the trio in one lockfile and run wallet rejection, wrong-chain, pending, replacement, revert, and typed-data tests after upgrades.
- **Misconfigured chain:** fail closed on unknown chain IDs and unverified contract addresses before displaying a sign action.
