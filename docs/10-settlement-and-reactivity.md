# 10. Settlement and reactivity

## Settlement authority

After a DreamDEX settlement finalization event, the worker reads the finalized payout vector and `voided` state from chain. For non-void markets, it derives the winning outcome as the unique maximum payout index and verifies the vector is valid. For void markets, it never fabricates a winner and routes to the documented refund/manual path.

## Bounded EIP-712 redeem

The SDK module exposes:

`redeemFor(address owner,uint256 nonce,uint256 deadline,bytes sig,uint32 operatorId,bytes32 venueId,bytes32 marketId,uint8 outcomeIdx,uint256 amount)`

Typed-data domain and struct must be generated from the active chain and deployed module address:

- domain name `SomniaMarkets`, version `1`;
- struct fields in order: `owner`, `operatorId`, `venueId`, `marketId`, `outcomeIdx`, `amount`, `nonce`, `deadline`;
- nonce and deadline are bounded, single-use, and persisted before submission.

The owner signs the typed data in the wallet. Oraclelane may submit the bounded redemption transaction only after signature validation and replay checks. It never creates an unbounded operator permission.

The public API sequence is `create redeem intent → browser signs via wagmi/viem → submit signature → worker queues bounded operator submission → position projection exposes status`. The operator receives only the bounded intent and owner signature required for that redemption; it has no authority to create a new market action.

## Reactivity posture

Reactivity is an optimization and hackathon proof, not the only settlement path. The worker must support:

1. event subscription/reactivity trigger;
2. idempotent finality confirmation;
3. bounded redeem submission;
4. retry with backoff and dead-letter state;
5. visible manual fallback with direct contract context.

If the reactive runtime is unavailable, the UI still exposes `Claim manually` and links to the transaction/market evidence.
