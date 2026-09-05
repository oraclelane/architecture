# 21. Screen handoff

## Radar

**Purpose:** help users find a live, understandable market. Desktop uses a bordered market list with a filter rail; mobile uses a single-column list.

> **Resolved, 2026-09-05.** Three documents disagreed about this surface: this file asked for a two-column card grid, `blueprint/docs/17-information-architecture-and-wireframes.md:50` for a three-column grid above 1100px, and the Slice 0 redesign note for a bordered list that "must not become a card grid". The implementation followed the list on `/radar` and cards on `/positions`, which is why the two pages read as different products. The split is now deliberate and stated: **radar is a list** — it is scanned, and rows compare well down a column — while **positions are cards**, because each is a self-contained holding rather than a row in a comparison. The two grid instructions are superseded.

```text
[Header: Oraclelane | Shannon | Connect wallet]
[Radar] [Freshness: updated 12s ago] [Refresh]
[Asset: All BTC ETH] [Status: Open] [Sort: Close time]
┌ Market card ───────────────────────┐
│ Will BTC be above $X at 18:00 UTC? │
│ UP  0.58      DOWN 0.42             │
│ Closes in 2h 14m  •  Fresh          │
│ [Read case file]                    │
└────────────────────────────────────┘
```

## Case file

Show the exact market question and terms before any AI content. Group chain facts (status, close/lock, outcomes, prices) separately from evidence and thesis. Primary CTA is `Build a thesis`; trading CTA remains disabled until the market is fresh and open.

## Thesis panel

Use a clear label `Oraclelane thesis · advisory`. Show direction, confidence band, three drivers maximum, risks, cited evidence, generated time, and expiry. `NO_TRADE` uses the same visual weight as directional outcomes and recommends waiting.

## Safety gate and trade drawer

The drawer is a review surface, not a shortcut. Show amount, selected outcome, max slippage, chain, contract target, quote expiry, and each deterministic check. `BLOCK` state explains the exact fix (e.g. “Market data is stale—refresh before trading”), with no override button.

## Positions and settlement

Use a vertical timeline with explicit timestamps and transaction links. Finalized non-void positions show `Winning outcome` and `Redeem`. Void positions show `Market void` and the documented refund/manual action. Auto-redeem states are `Queued`, `Signature required`, `Submitted`, `Confirmed`, or `Manual action required`.

## Responsive behavior

| Breakpoint | Behavior |
|---|---|
| `≥ 1200px` | filter rail + two-column content; activity rail visible |
| `768–1199px` | filter collapses to toolbar; one/two adaptive columns |
| `< 768px` | single column; drawers become bottom sheets; sticky primary CTA |

The mobile layout never hides contract/expiry information; it may move it into an always-available review section.
