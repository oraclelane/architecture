# 18. UI/UX design brief

This brief is the handoff from technical architecture to visual/product design. Figma becomes the visual source of truth; this repository remains the technical source of truth.

## Primary journey

`Radar → Case File → Thesis → Safety Gate → Wallet Sign → Position → Settlement`

The design should make the trust model visible: what came from chain data, what came from external evidence, what the AI suggests, and what the deterministic gate permits.

## Required screens

1. Market Radar: filterable BTC/ETH markets, price, close time, freshness, liquidity signal.
2. Case File: exact question/terms, outcomes, close/lock timing, source freshness, trading CTA.
3. Thesis Panel: direction, confidence band, concise rationale, evidence list, timestamp, expiry, “not financial advice” copy.
4. Safety Gate: passed/failed checks, amount/slippage inputs, blocked reason, re-quote action.
5. Wallet Drawer: chain, contract, outcome, amount, expiry, simulation/allowance status, sign button.
6. Position Timeline: pending/open/locked/finalized/redeemed states with tx links.
7. Settlement: winning outcome or void state, auto-redeem progress, manual fallback CTA.

## State coverage

For each screen provide desktop/mobile variants and loading, empty, stale, unavailable, blocked, pending, success, failure, and wallet-rejected states. Do not hide uncertainty behind a spinner or generic “something went wrong” message.

## Design acceptance criteria

- A new user can explain why a trade is allowed or blocked without reading documentation.
- AI advice is visually distinct from chain facts and never styled as a guarantee.
- The user sees the exact material transaction fields before signing.
- Manual settlement remains discoverable after any automation failure.
- Every interactive state maps to a frozen API enum or documented local presentation state.
