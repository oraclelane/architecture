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
