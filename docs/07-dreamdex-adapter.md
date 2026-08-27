# 7. DreamDEX adapter

The adapter is the only module allowed to know DreamDEX SDK method names, contract addresses, ABI quirks, and event decoding. Pin the SDK version and record the exact version in the implementation repository.

## Protocol facts to preserve

- Somnia networks: Shannon `50312`, Elwood `50313`, mainnet `5031`.
- Settlement v3 uses a payout vector. Do not call the removed/reverting `winningOutcome()` view. Derive the winning index only after finalized/resolved; treat `voided=true` separately.
- There are two `MarketFinalized` event shapes: module-level `(bytes32 indexed marketId,address indexed pool,uint256 marketKey)` and settlement-level `(uint256 indexed marketKey,address indexed pool,uint64 nonce,address collateralToken,uint256 netBacking,bool voided,uint8 winningOutcome)`.
- `outcomeToken` is resolved from market/settlement state; it is not a static global configuration value.
- Current SDK addresses must be environment-configured and verified at startup; never hardcode an address in frontend code.

## Adapter responsibilities

1. Fetch market metadata and normalize lifecycle/status.
2. Read pool balances/prices and preserve integer precision.
3. Resolve outcome-token addresses for a specific market and outcome index.
4. Decode both finalization events and correlate by chain, pool, and market key.
5. Encode trade and redeem calldata using the pinned SDK.
6. Expose freshness, block number, and source timestamp with every snapshot.

## Staleness policy

`FRESH` means observed within the configured TTL and from a healthy upstream. `STALE` may render but cannot pass the trade safety gate. `UNKNOWN` is an operational error; the UI must offer retry/manual navigation.
