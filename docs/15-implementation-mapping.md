# 15. Implementation mapping

This is the bridge from architecture to the future `oraclelane/app` repository.

| Slice | Contract/artifact | Owner boundary | Acceptance evidence |
|---|---|---|---|
| Market radar | `GET /markets` | MarketPort + web query | live Shannon cards with freshness |
| Case file | `GET /markets/{id}` | snapshot mapper | terms/outcomes/close state match chain |
| Thesis panel | `POST /theses` | ThesisPort | citations validate; expiry visible |
| Safety gate | `POST /trade-previews` | domain policy | stale/closed/limit cases block |
| Wallet trade | transaction observation | browser wallet | user signs; receipt reconciles |
| Positions | `GET /positions` | projection repository | tx-derived lifecycle |
| Settlement | worker/event adapter | SettlementPort | finalized + void fixtures |
| Redeem | redeem-intent + EIP-712 | wallet/operator boundary | nonce/deadline/replay tests |
| Reactivity | event trigger | worker runtime | auto path + manual fallback |

## Definition of ready

A slice is ready for UI integration when its OpenAPI schemas, examples, errors, mock fixtures, provider contract tests, and one happy-path trace exist. A slice is demo-ready only when the testnet evidence uses the same release SHA as the shown UI and docs.

## Future repository mapping

```text
apps/web                 # presentation and wallet UX
apps/api                 # HTTP controllers and application services
apps/worker              # settlement/reactivity jobs
packages/domain          # pure policies and state machines
packages/contracts      # generated OpenAPI/event types
packages/dreamdex        # pinned SDK/RPC adapter
packages/ui              # design-system implementation
contracts/              # only if a future Oraclelane contract is required
```
