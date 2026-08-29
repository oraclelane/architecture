# 26. Market Case File wireframe handoff

**Status:** Ready for Figma exploration; pending ADR-0009 contract decision  
**Target route:** `/markets/:chainId/:marketId`  
**Primary viewport:** desktop `1440 × 1024`  
**Responsive viewport:** mobile `390 × 844`  
**Source of truth:** this handoff for behavior and information hierarchy; [`25-market-case-file-design.md`](25-market-case-file-design.md) for product scope and states; `20-design-system.md` for tokens

## Overview

This screen is a read-only Case File for one DreamDEX Event Contract. It is reached from a Radar card and gives a user enough context to understand market validity, outcomes, timing, resolution terms, and evidence before any thesis or execution flow appears.

Slice 1 does not connect a wallet, request an AI thesis, calculate payout, or submit a transaction. The final control is a deferred next-step affordance, not a trade CTA.

## Figma file structure

Create one Figma page named `Case File / Slice 1` with these frames:

1. `Desktop / open-fresh`
2. `Desktop / open-stale`
3. `Desktop / locked`
4. `Desktop / terms-unavailable`
5. `Desktop / evidence-unavailable`
6. `Desktop / loading`
7. `Desktop / not-found`
8. `Desktop / upstream-error`
9. `Mobile / open-fresh`
10. `Mobile / open-stale`
11. `Mobile / locked`
12. `Mobile / terms-unavailable`

Use the existing Oraclelane shell, sidebar, typography, and neutral shadcn tokens. Do not introduce a new palette or a second set of radii.

## Layout specification

### Desktop

| Region | Specification |
|---|---|
| App shell | Existing shell from Slice 0; preserve sidebar/header behavior. |
| Content max width | Use the project container token and existing `max-w` convention; target a comfortable reading width rather than edge-to-edge content. |
| Page padding | Use the documented spacing scale; no arbitrary one-off values. |
| Header row | Back control at start; title/question summary and status grouped; refresh and freshness at end. |
| Question card | Full-width, first reading block, visually dominant. |
| Primary grid | Two columns for outcomes and timing/health at desktop; stack below the tablet breakpoint. |
| Terms | Full-width card below primary grid. |
| Evidence | Full-width card below terms. |
| Next step | Secondary/deferred control below evidence; no sticky trade action. |

Recommended desktop hierarchy:

```text
Shell
└── Context header
    ├── Back to Radar
    ├── Market summary + status
    └── Freshness + Refresh
└── Question card
└── Two-column facts grid
    ├── Outcomes card
    └── Timing and health card
└── Resolution terms card
└── Evidence and provenance card
└── Deferred next-step control
```

### Mobile

At `<768px`, use a single-column flow with safe-area padding:

```text
Back + title + status
Freshness + Refresh
Question
Stale/locked alert (when applicable)
Outcome cards
Timing and health tiles
Terms disclosure
Evidence disclosure
Deferred next step
```

Terms and evidence may collapse to reduce scroll length, but stale/locked warnings and the exact question must remain visible without interaction. Do not hide contract validity behind a menu.

## Component inventory

| Component | Variant | Required props/content | Behavior |
|---|---|---|---|
| `Breadcrumb` or back `Button` | `ghost` | `Back to Radar` | Returns to Radar and preserves URL-safe filters. |
| `Card` | default | Question, outcomes, timing, terms, evidence | Use header/content/footer composition. |
| `Badge` | neutral/status | Market status | Text and icon; never color alone. |
| `Badge` | secondary/freshness | `Fresh`, `Stale`, or `Unknown` | Stale/unknown has explanatory copy nearby. |
| `Alert` | neutral/destructive | Stale, locked, unavailable, or provider error | Includes impact and next action. |
| `Skeleton` | default | Header, question, outcome, terms rows | Match final geometry to limit layout shift. |
| `Accordion`/`Collapsible` | default | Terms and evidence on mobile | Keyboard operable; warnings remain expanded. |
| `Separator` | default | Between fact groups | Maintains grouping in dense layouts. |
| `Tooltip` | default | Localized timestamp or advanced ID | Full UTC value remains accessible. |
| `Button` | `secondary`/`outline` | `Refresh`, deferred next step | Refresh has loading state; next step is disabled/deferred. |
| `Empty` | default | Not found or no evidence | Gives a recoverable route back to Radar. |

## Content and copy handoff

### Header copy

- Back: `Back to Radar`
- Fresh: `Fresh · observed {relative time}`
- Stale: `Stale · last observed {absolute UTC}`
- Unknown: `Freshness unavailable`
- Refresh loading: `Refreshing market data…`
- Refresh failure: `Could not refresh this market. Showing the last valid observation.`

### Status copy

| Status | Label | Supporting explanation |
|---|---|---|
| `OPEN` | `Open` | `Market is open according to the latest observation.` |
| `PAUSED` | `Paused` | `Trading state is paused; review only.` |
| `LOCKED` | `Locked` | `Market is locked; no new action is available.` |
| `RESOLVED` | `Resolved` | `Market outcome has been finalized.` |
| `VOID` | `Void` | `Market is void; see the published resolution terms.` |

### Unavailable copy

Use concrete field names in supporting text:

- `Resolution terms unavailable from the current source.`
- `Lock time unavailable; it is not inferred from close time.`
- `Evidence unavailable from the current source.`
- `Canonical contract link unavailable.`

Avoid generic copy such as `Something went wrong` when the missing capability is known.

### Question and terms

- Render the exact question without paraphrasing.
- Allow long questions to wrap naturally; never truncate the decision sentence with an ellipsis.
- Terms fields may use up to three lines in the collapsed summary, followed by `Show full terms`.
- Preserve source wording for resolution rules; any plain-language explanation must be clearly labeled as an explanation.

### Outcome copy

- Show the exact outcome label and price string.
- Keep token address and outcome index in an `Advanced details` disclosure.
- Do not show `potential payout`, `profit`, `probability`, or `spread` in Slice 1.

## Interaction states

| Element | State | Specification |
|---|---|---|
| Back | Hover/focus/pressed | Use semantic shadcn state; focus ring is always visible. |
| Refresh | Idle | Requests the same market and keeps current content. |
| Refresh | Loading | Disable only refresh; show spinner and live-region status. |
| Refresh | Success | Replace content after schema validation; update observed/freshness copy. |
| Refresh | Failure | Preserve last valid content; show alert with retry. |
| Terms disclosure | Closed | Show summary and `Show full terms`. |
| Terms disclosure | Open | Reveal all available terms; focus remains on disclosure control. |
| Evidence link | Hover/focus | Explain external navigation; open safely in a new tab only if product policy permits. |
| Deferred next step | Disabled | Explain `Available in the next slice`; never imply wallet or trade readiness. |

## Responsive behavior

| Breakpoint | Behavior |
|---|---|
| `≥1200px` | Two-column facts grid; full terms/evidence cards; existing desktop shell. |
| `768–1199px` | Single main column or adaptive two-column facts grid; sidebar/filter context may collapse per shell rules. |
| `<768px` | Single column; terms/evidence disclosures; no horizontal overflow; stale/locked alert stays expanded. |

Long outcome labels, long source titles, and translated text must wrap without changing the meaning or pushing the refresh control off-screen.

## Motion and transitions

Use the existing motion guidance:

- hover/focus: `150ms ease-out`;
- disclosure/sheet-like expansion: `220ms ease-out`;
- refresh spinner: continuous only while the request is active;
- no celebration animation or profit-oriented visual feedback;
- respect `prefers-reduced-motion` by removing non-essential transitions.

## Accessibility handoff

- One `h1` contains the exact market question or a stable, source-derived short title.
- Use landmark regions for header, main content, terms, evidence, and status alerts.
- Focus order: back → summary/status → freshness/refresh → question → outcomes → timing → terms → evidence → next step.
- Refresh updates a polite live region and exposes `aria-busy` on the content region while loading.
- Status and freshness badges include text labels; icons are supplementary.
- Disclosure controls expose `aria-expanded` and `aria-controls`.
- External links have an accessible label indicating destination behavior.
- Advanced IDs remain copyable/selectable and use a readable monospace face.
- Contrast meets WCAG AA in both light and dark themes.

## Prototype data requirements

Figma annotations should label every value as one of:

1. **Contract fact** — from the typed Market or proposed Case File contract.
2. **Provider evidence** — from the evidence envelope and source metadata.
3. **Presentation state** — local UI state such as loading, stale alert, or disclosure open.
4. **Deferred capability** — visible only as unavailable/deferred copy.

Use the existing fixture names and add these Case File-specific variants after ADR-0009 is accepted:

- `market-case-file-open-fresh`;
- `market-case-file-open-stale`;
- `market-case-file-locked`;
- `market-case-file-terms-unavailable`;
- `market-case-file-evidence-unavailable`;
- `market-case-file-not-found`;
- `market-case-file-upstream-error`.

## Handoff checklist

- [ ] Desktop and mobile frames exist for every required state.
- [ ] No frame shows an active wallet, trade, or redeem action.
- [ ] Exact question, status, freshness, outcomes, close time, and unavailable behavior are annotated.
- [ ] Terms, lock timing, evidence, and contract-link fields are marked as pending contract decision where applicable.
- [ ] Components reference shadcn semantic tokens and the existing radius/spacing scale.
- [ ] Keyboard focus order and screen-reader labels are documented in Figma notes.
- [ ] Figma copy matches the state matrix in `25-market-case-file-design.md`.
- [ ] Contract fixtures are not created until the additive detail schema is accepted.

## Engineering handoff outcome

When the contract decision and these frames are approved, the implementation team can create Slice 1 without guessing about page hierarchy, responsive behavior, unavailable data, or execution boundaries. The next engineering plan should cover only the approved detail contract, typed fetcher, fixtures, and read-only Case File surface.
