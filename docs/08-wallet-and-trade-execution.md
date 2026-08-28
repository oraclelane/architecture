# 8. Wallet and trade execution

## Non-custodial sequence

1. Browser requests a preview with owner, market, outcome, amount, and slippage.
2. API re-reads chain state, evaluates deterministic checks, and returns calldata plus an expiry.
3. Browser displays every material field: chain, contract, outcome, amount, max slippage, expiry, and gas estimate.
4. User explicitly signs/submits through the wallet provider.
5. Browser sends only the transaction hash to the API.
6. Worker/API observes receipt and events; projection moves from `PENDING` to `OPEN` or `FAILED`.

Wallet sessions use a one-time API challenge signed by the owner. Every owner-scoped mutation requires the resulting expiring bearer session and checks that the session address matches the resource owner.

## Deterministic safety gate

All must pass for `ALLOW`:

- supported chain and verified contract addresses;
- market status `OPEN`, not stale, not past close/lock;
- outcome index exists and token resolves;
- amount within per-trade and session budget;
- balance/allowance sufficient;
- quoted price and slippage within user limits;
- quote and thesis (if shown) are unexpired;
- no duplicate pending transaction for the same intent.

`WARN` is display-only and requires explicit user confirmation. `BLOCK` cannot be overridden by the UI.

## Wallet threat controls

Use checksum validation, chain-switch prompts, transaction simulation where available, and an allowlist of DreamDEX contracts per environment. Never ask for seed phrases or private keys. Never make an approval broader than the exact token/spender requirement.
