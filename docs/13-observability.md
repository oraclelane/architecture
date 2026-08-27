# 13. Observability

## Correlation

Every HTTP request, thesis run, quote, wallet tx, settlement event, and redeem attempt has a `requestId`/`traceId`. Chain-bound records include `chainId`, block number, tx hash, market key, and release SHA.

## Metrics

- `market_snapshot_freshness_seconds`
- `thesis_latency_ms`, `thesis_validation_failures_total`
- `trade_preview_allow_total`, `trade_preview_block_total{reason}`
- `wallet_tx_confirmed_total`, `wallet_tx_failed_total`
- `settlement_events_total{voided}`
- `redeem_attempt_total`, `redeem_success_total`, `redeem_manual_fallback_total`
- `rpc_error_total`, `indexer_lag_seconds`

## Alerts

Alert on stale market data, elevated quote conflicts, settlement projection lag, repeated redeem failures, worker dead-letter growth, and unexpected contract/address mismatch. Do not alert on normal user wallet rejection.

## Judge-visible evidence

The demo can expose a sanitized activity timeline: exact market ID, chain, thesis evidence timestamp, safety decision, user tx hash, finalization event, and redemption/manual fallback state. Secrets and raw prompts remain private.
