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
4. Session activity log: latest wallet/settlement events, opened from a control in the header.

> **Amended, 2026-09-05.** This was a *persistent* rail docked beside the page on desktop. It cost every route a 240px column plus its gap at all widths above 1180px, and because the row aligned to the top, the space below the rail could not be reclaimed — so all seven routes rendered into roughly 1128px of a 1440px window and left the rest empty. The log marks but never alerts, and it clears on reload, which is the weakest claim on permanent space in the interface. It is now summoned from the header, with a mark on the trigger when it holds something actionable, so the signal survives a closed panel. One behaviour at every width; the rule that hid it below 1180px is gone with it.
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
