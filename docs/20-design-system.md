# 20. Oraclelane design system

The visual direction is **calm terminal clarity**: high information density without casino-like urgency. The interface should feel like a verification console, not a slot machine.

## Tokens

| Token | Value | Usage |
|---|---|---|
| `color.canvas` | `#07111F` | application background |
| `color.surface` | `#0E1B2D` | cards and drawers |
| `color.surfaceRaised` | `#14243A` | hover/active surfaces |
| `color.text` | `#F4F7FB` | primary text |
| `color.textMuted` | `#9BAAC0` | secondary text |
| `color.brand` | `#55D6BE` | confirmed/primary action |
| `color.info` | `#78A8FF` | links and evidence |
| `color.warning` | `#F4C95D` | uncertainty/stale |
| `color.danger` | `#FF7A7A` | blocked/failed |
| `color.border` | `#223551` | separators |
| `space-1..6` | `4, 8, 12, 16, 24, 32px` | layout scale |
| `radius-sm/md/lg` | `6, 10, 16px` | controls/cards |

Typography uses a readable sans-serif (Inter or system fallback) and a monospace face for addresses, IDs, amounts, and transaction hashes. Use tabular numerals for prices and times.

## Components

| Component | Variants | Required states |
|---|---|---|
| `MarketCard` | default, compact | fresh, stale, locked, resolved, skeleton |
| `EvidenceList` | inline, expanded | loading, empty, source unavailable |
| `ThesisPanel` | expanded, compact | valid, expired, unavailable, rejected |
| `SafetyGate` | allow, warn, block | evaluating, blocked, re-quote |
| `TradeDrawer` | desktop drawer, mobile sheet | editing, signing, pending, rejected, failed |
| `PositionTimeline` | card, detail | pending, open, locked, finalized, void, redeemed |
| `StatusBadge` | semantic | never rely on color alone; include text/icon |
| `CopyableValue` | address, tx hash | copied, unavailable |

## Accessibility

- WCAG AA contrast target for text and controls.
- Keyboard order follows the task flow; focus moves into drawers and returns to the trigger.
- Every status badge has text and an accessible label.
- Tables/cards expose equivalent semantics to screen readers.
- `prefers-reduced-motion` disables non-essential transitions.

## Motion

Use 150ms ease-out for hover/focus, 220ms ease-out for drawer transitions, and no looping animation for financial states. A pending transaction uses a static status plus optional subtle progress indicator.
