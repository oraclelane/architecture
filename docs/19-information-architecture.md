# 19. Information architecture

## Navigation model

```text
Oraclelane
├── Radar (default)
│   ├── All markets
│   ├── BTC
│   └── ETH
├── Positions
│   ├── Active
│   ├── Awaiting settlement
│   └── Ready to redeem
└── Help / Trust
    ├── How Oraclelane works
    ├── AI and evidence policy
    └── Contract and transaction links
```

The app has one primary job: help a user understand and safely act on an existing Event Contract. Market creation, social feeds, leverage, and portfolio strategy are intentionally absent from MVP navigation.

## Core task flows

### First-time trader

`Radar → select market → read terms → inspect evidence → choose amount → review safety checks → connect wallet → sign → view position`

### Returning trader

`Radar → Positions → inspect lifecycle → settle/redeem or open market`

### Automation failure

`Positions → Ready to redeem → read settlement proof → sign/manual redeem → confirmation`

## Page hierarchy

1. Global header: Oraclelane mark, network badge, wallet status, help.
2. Context bar: current page title, freshness timestamp, refresh action.
3. Main content: cards/tables; no important state is hidden behind hover.
4. Persistent activity rail on desktop: latest wallet/settlement events.
5. Mobile bottom navigation: Radar, Positions, Help; trade actions remain in the content flow.

## URL/state conventions

- `/radar`
- `/markets/:chainId/:marketId`
- `/positions`
- `/positions/:positionId`
- `/help/trust`

Filters, selected outcome, and amount are URL-safe local state; wallet signatures and transaction data are never placed in query strings.

## Reconciliation with blueprint's IA

`blueprint/docs/17-information-architecture-and-wireframes.md` defines an overlapping but not identical IA: it adds a `/` landing route (product explanation before wallet connect) and a `/settings` route (risk preferences and authorizations) that this document omits, and uses `/markets` where this document uses `/radar`. The actual Figma design merges both: this document's screen structure and route shape as the backbone, plus blueprint's `/` and `/settings` additions. Treat the merged set — `/`, `/radar`, `/markets/:chainId/:marketId`, `/positions`, `/positions/:positionId`, `/settings`, `/help/trust` — as current until this document and blueprint's are fully unified.
