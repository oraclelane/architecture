# 14. Failure and recovery

| Failure | User-visible state | Retry | Recovery |
|---|---|---|---|
| RPC timeout | data stale / retry | exponential, max 3 | alternate RPC or manual refresh |
| Indexer lag | `UNKNOWN` freshness | poll chain | disable trade gate until fresh |
| LLM timeout | thesis unavailable | one bounded retry | continue without AI |
| Invalid thesis/citation | thesis rejected | no blind retry | show unavailable, log validator |
| Quote expired | re-quote required | none | user confirms new preview |
| Wallet rejected | signature rejected | user-controlled | preserve no-position state |
| Tx reverted | trade failed | no automatic duplicate | show receipt and reason |
| Finality event duplicated | no state change | idempotent consume | audit duplicate |
| Redeem submit fails | manual action required | bounded retry | wallet/manual redeem CTA |
| DB unavailable | degraded API | retry connection | rebuild projections from chain |

## Recovery rules

- Never retry a user transaction blindly after an unknown receipt; reconcile by hash first.
- Use an outbox/inbox or unique event key for worker idempotency.
- Mark poison events dead-lettered with operator context; do not silently drop.
- Reconciliation can move a projection forward from chain truth but never backward without an explicit correction record.
