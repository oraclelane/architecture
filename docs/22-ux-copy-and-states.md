# 22. UX copy and state matrix

## Voice

Use precise, calm, non-promotional language. Prefer “The market is stale, so trading is paused” over “Oops!” Avoid promising profit or implying that confidence equals probability of winning.

## Canonical copy

| Context | Copy |
|---|---|
| AI label | `Oraclelane thesis · advisory, not a guarantee` |
| Stale market | `Market data is stale. Refresh before reviewing a trade.` |
| Blocked gate | `Trade blocked until the checks below pass.` |
| Wallet prompt | `Review these details in your wallet before signing.` |
| Pending tx | `Submitted. Waiting for Somnia confirmation.` |
| AI unavailable | `Thesis unavailable right now. Market facts and manual trading remain available.` |
| Auto-redeem failed | `Automatic redemption needs attention. You can continue manually.` |
| Void market | `This market was voided. No winning outcome is assigned.` |

## State matrix

| Surface | Loading | Empty | Error/degraded | Success |
|---|---|---|---|---|
| Radar | skeleton cards | `No open markets match this filter.` | stale banner + retry | fresh cards |
| Case file | fact skeleton | market not found | upstream unavailable | terms and outcomes |
| Thesis | evidence skeleton | not requested | unavailable/rejected | validated thesis + expiry |
| Safety gate | `Checking live conditions…` | n/a | stale/quote conflict | allow/warn/block checks |
| Wallet | disabled until preview | n/a | rejected/failed | tx hash + pending |
| Positions | timeline skeleton | `No positions yet.` | projection delayed | reconciled lifecycle |
| Settlement | finality pending | n/a | automation/manual fallback | redeemed or void |

## Content limits

- Market question: two lines on card, full text on case file.
- Thesis summary: 1200 characters maximum; show drivers/risks as short bullets.
- Evidence title: two lines, then tooltip/expanded view.
- Address/hash: truncated visually but copyable; never truncate the accessible label.
- Error detail: one actionable sentence in UI; correlation ID available for support.
