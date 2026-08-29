# 25. Market Case File design

**Status:** Proposed Slice 1 design  
**Owner:** Oraclelane product and frontend team  
**Depends on:** Slice 0 foundation + radar preview, `GET /markets/{marketId}`, and the canonical shadcn token system  
**Out of scope:** wallet connection, trade execution, signing, redeem, autonomous entry, market creation, and AI thesis generation

## 1. Product intent

The Market Case File is the decision-quality detail view for one DreamDEX Event Contract. It turns a compact Radar card into a verifiable explanation of what the market asks, how it resolves, what data is current, and which information is unavailable.

The Case File is not a trading screen in Slice 1. Its job is to reduce avoidable confusion before a user proceeds to the later thesis, safety, and wallet slices.

### User problem

Radar cards are intentionally compact. A user still needs to answer four questions before considering a position:

1. What exactly is the event question?
2. What are the possible outcomes and current prices?
3. When does trading close or lock, and how fresh is the observation?
4. Which resolution terms and evidence can be verified right now?

If any answer is unavailable, the interface must say so plainly. It must never fill a contract gap with an inferred rule, stale number, or AI-generated claim.

### Product hypothesis

If users can inspect a precise, calm, and source-aware Case File before seeing a trade action, they will understand Event Contracts faster, trust the DreamDEX integration more, and be more likely to continue into a later thesis and safety flow.

## 2. Scope boundary

### Included in Slice 1

- Route `/markets/:chainId/:marketId`.
- Navigation from a Radar card and a keyboard-accessible back path.
- Exact market question and plain-language metadata.
- Outcome labels and price strings without floating-point conversion.
- Market status, close timing, and observed/freshness metadata.
- Resolution terms and evidence section when supplied by the frozen contract.
- Explicit unavailable, stale, locked, not-found, and upstream-error states.
- Refresh that preserves the last valid response until a replacement succeeds.
- A disabled or clearly deferred next-step control for the later thesis/trade flow.

### Explicitly excluded

- Wallet connection or network switching.
- `POST /theses` and AI direction/confidence.
- `POST /trade-previews`, amount entry, slippage, or safety decisions.
- Transaction signing, position creation, settlement, or redeem.
- Market creation, social feeds, leverage, copy trading, or pooled custody.

The boundary is deliberate: a detailed market explanation can be demoed and validated without introducing signing risk or pretending that a missing execution path is complete.

## 3. Entry points and user flow

### Primary flow

```text
/radar
  → choose a market card
  → /markets/:chainId/:marketId
  → inspect validity, outcomes, timing, and evidence
  → refresh if stale
  → return to Radar or continue when the next slice is available
```

### Entry-point rules

| Entry point | Behavior |
|---|---|
| Radar card | Opens the Case File using the card's `chainId` and `marketId`. |
| Direct URL | Loads the detail resource; an unknown market shows a recoverable not-found state with a Radar link. |
| Browser back | Returns to Radar while preserving URL-safe filters. |
| Refresh action | Requests the same market and retains the last valid content during loading. |
| Locked/stale card | May open the Case File for inspection but must not expose an active trade CTA. |

## 4. Information architecture

The page follows the trust order defined by the existing IA:

```text
Market validity → Contract terms → Evidence freshness → User decision → Next step
```

### Desktop anatomy

```text
+-----------------------------------------------------------------------+
| ← Back to Radar       BTC above $100,000 by Dec 31       [OPEN]        |
| Chain badge · observed 12s ago                         [Refresh]       |
+-----------------------------------------------------------------------+
|                                                                       |
| Exact market question                                                  |
| What is the settlement condition?                                     |
|                                                                       |
| +------------------------------+  +-------------------------------+  |
| | Outcomes and prices          |  | Timing and market health       |  |
| | YES  $0.56                   |  | Closes 31 Dec 2026 18:00 UTC  |  |
| | NO   $0.45                   |  | Status: Open                   |  |
| | Prices are decimal strings   |  | Freshness: Fresh               |  |
| +------------------------------+  +-------------------------------+  |
|                                                                       |
| Resolution terms                                                      |
| Source, rule, cutoff, and void behavior                               |
|                                                                       |
| Evidence and data provenance                                          |
| [source rows, published time, relevance, source freshness]           |
|                                                                       |
| [Read thesis — available in next slice]                               |
+-----------------------------------------------------------------------+
```

The primary visual weight belongs to the exact question and market validity. Price is context, not a promise of payout. The future trade action is not presented as available in Slice 1.

### Mobile anatomy

Below `768px`:

1. Back, title, status, and freshness remain visible at the top.
2. The exact question is the first content block.
3. Outcome cards stack vertically and retain full labels.
4. Timing and health use two-column metric tiles where space permits.
5. Terms and evidence use disclosure sections, but stale/locked warnings stay expanded.
6. The deferred next-step control remains reachable after the evidence section; it must not become a fake sticky trade button.

## 5. Content specification

### Header

| Element | Required behavior |
|---|---|
| Back control | Label includes destination, for example `Back to Radar`; keyboard focus returns predictably. |
| Market title | Uses a short human-readable label only when it is derived from known data; do not invent a title that changes the question. |
| Status badge | Uses semantic text (`Open`, `Paused`, `Locked`, `Resolved`, `Void`) and an icon/label, never color alone. |
| Chain badge | Displays the selected `chainId` using a documented network label. Unknown chains are shown as `Unknown network`, not silently remapped. |
| Freshness | Shows relative time plus an accessible absolute UTC value. `UNKNOWN` freshness remains explicit. |
| Refresh | Uses `secondary`/`outline` styling; disables only itself while loading. |

### Question and terms

The question is rendered verbatim from the trusted contract field. Terms must distinguish:

- resolution source;
- resolution rule;
- cutoff/close and lock behavior;
- allowed outcomes;
- void or unavailable resolution behavior;
- a link to the canonical contract or source when available.

Terms are not paraphrased into a stronger claim than the source. If a required term is absent, show `Resolution terms unavailable from the current source` and block the next-step control.

### Outcomes and prices

Each outcome row contains the outcome index, exact label, price string, and token address only in an advanced disclosure. Price display uses tabular numerals and preserves the original decimal string. The UI must not calculate payout, probability, spread, or profit in Slice 1.

Outcome direction is communicated with text and iconography. Green/red alone is prohibited, and a higher price is not labeled as a guaranteed winner.

### Timing and health

Display:

- close time in UTC and localized time via tooltip;
- lock time only when the API supplies it;
- current status;
- observed-at timestamp;
- freshness state and its impact.

If `lockAt` is not available, the page must say `Lock time unavailable` rather than deriving it from `closeAt`.

### Evidence and provenance

Evidence is a provenance surface, not an AI recommendation. Each row should show title, source type/relevance, published time, and a safe external link when the frozen contract supplies those fields. The page distinguishes:

- on-chain/market facts;
- external evidence;
- future AI-generated thesis.

AI content is not rendered in Slice 1. The later Thesis slice must use its own advisory treatment and expiry.

## 6. Contract mapping and readiness

The current OpenAPI `Market` schema provides the following fields for Slice 1:

| UI need | Current field | Handling |
|---|---|---|
| Market identity | `marketId`, `chainId` | Required; preserve as strings/integers. |
| Exact question | `question` | Render verbatim. |
| Status | `status` | Map one-to-one to documented enum. |
| Outcomes | `outcomes[].index`, `label`, `price`, `tokenAddress` | Display label and price; keep address advanced. |
| Close time | `closeAt` | Display with UTC and local affordance. |
| Observation time | `observedAt` | Display absolute and relative time. |
| Freshness | `freshness` | Drive fresh/stale/unknown copy and CTA policy. |

The current schema does **not** yet contain the fields required for a complete Case File:

| Required capability | Current gap | Decision gate before implementation |
|---|---|---|
| Resolution terms | No `terms`/resolution object | Confirm DreamDEX source and freeze an additive schema. |
| Lock timing | No `lockAt` | Confirm whether DreamDEX exposes lock separately from close. |
| Evidence | No evidence collection on `Market`/`MarketResponse` | Confirm provider, provenance, URL safety, and freshness semantics. |
| Canonical contract link | No contract/source URL | Confirm whether this is derived from chain metadata or returned by the adapter. |

Slice 1 implementation must not invent these fields in the client. The backend/API contract must either add them or explicitly return a typed unavailable state. Until that decision is frozen, the UI design uses unavailable placeholders and keeps the deferred next-step control inactive.

## 7. State matrix

| State | User-visible treatment | Allowed actions |
|---|---|---|
| Loading | Skeleton for header, question, outcome rows, and terms; accessible busy label. | Cancel via back; no refresh duplication. |
| Open + fresh | Full content with `Open` and `Fresh` labels. | Refresh; inspect terms/evidence; deferred next step. |
| Open + stale | Persistent stale alert with observed timestamp and impact. | Refresh; inspect last valid content; no active trade action. |
| Paused | Status explanation and no execution language. | Refresh; return to Radar. |
| Locked | Locked alert and read-only outcomes/terms. | Refresh; return to Radar. |
| Resolved | Resolution status; no trade language. | Inspect terms/evidence; return to Radar. |
| Void | Void status and explicit explanation if supplied. | Inspect; return to Radar. |
| Unknown freshness | `Freshness unavailable`; do not imply currentness. | Refresh; inspect read-only content. |
| Missing required terms/evidence | Inline unavailable message naming the missing field. | Refresh; return to Radar; next step disabled. |
| Not found | Recoverable empty state with market ID and Radar link. | Return to Radar; retry. |
| Upstream unavailable | Error alert that preserves prior content when available. | Retry; return to Radar. |

Refresh behavior must be non-destructive: retain the last valid response while loading and replace it only after the new response passes schema validation. A failed refresh must expose the failure without erasing trustworthy prior data.

## 8. Component and token guidance

Use the existing shadcn composition from `20-design-system.md`:

- `Card` for question, outcomes, timing, terms, and evidence sections;
- `Badge` for status and freshness;
- `Alert` for stale, locked, unavailable, and error messaging;
- `Skeleton` for loading;
- `Accordion` or `Collapsible` for mobile terms/evidence;
- `Button` with `secondary`, `outline`, or `ghost` variants for back, refresh, and copy;
- `Separator` to reinforce information grouping;
- `Tooltip` for localized timestamps and advanced identifiers.

Semantic rules remain strict: facts use neutral/muted tokens; destructive is reserved for blocked/error meaning; no custom success/warning palette is introduced; dark mode remains the default demo presentation.

## 9. Accessibility and trust requirements

- The page has one `h1` containing the market question or a stable title derived from it.
- Status, freshness, and unavailable content are announced with text and accessible labels.
- Focus order follows header → question → outcomes → timing → terms → evidence → next step.
- Refresh exposes `aria-busy`/live-region feedback without moving focus unexpectedly.
- External evidence links identify that they open another site and never render unsanitized HTML.
- Prices, IDs, and timestamps use readable tabular/monospace presentation without sacrificing zoom or contrast.
- `prefers-reduced-motion` disables non-essential transitions.
- Stale and locked warnings are not hidden behind hover, menus, or collapsed sections.

## 10. Analytics and ecosystem signals

Instrumentation is proposed for product learning, not for making financial claims:

| Event | Purpose | Privacy rule |
|---|---|---|
| `case_file_opened` | Measure Radar-to-detail conversion. | Record market/chain identifiers, not wallet identity. |
| `case_file_refreshed` | Measure demand for freshness. | Record result state and latency. |
| `case_file_evidence_opened` | Measure trust/provenance usage. | Record source ID, not page contents. |
| `case_file_next_step_clicked` | Measure intent for the next slice. | Do not imply a trade occurred. |
| `case_file_error_seen` | Find provider/schema reliability issues. | Redact URLs and wallet data. |

These signals help demonstrate ecosystem impact to judges: users are not only viewing markets; they are returning to verify market state and inspecting the information that makes Event Contracts understandable.

## 11. Acceptance criteria

### Product

- A new user can explain the market question, outcomes, close timing, status, and freshness without opening developer documentation.
- The page clearly separates contract facts, external evidence, and future AI advice.
- No action on the page implies that a wallet is connected or a transaction will execute.

### Contract and correctness

- The detail route uses the typed `GET /markets/{marketId}` client with the required `chainId`.
- IDs, token addresses, prices, and timestamps are not converted through binary floating-point math.
- Missing terms, lock time, evidence, or canonical links are typed and visible as unavailable states.
- Stale/locked/resolved/void markets cannot expose an active trade CTA.

### Interaction and accessibility

- Loading, stale, unavailable, not-found, and upstream-error states are covered by fixtures.
- Refresh preserves the last valid response and reports failures accessibly.
- Desktop and mobile layouts remain usable at the project breakpoints with no horizontal overflow.
- Keyboard navigation and screen-reader labels cover every interactive element.

### Demo readiness

- The demo can open a Radar market, inspect its Case File, show a fresh-to-stale transition, refresh successfully, and show a locked read-only state.
- The demo explicitly calls out which capabilities are deferred to later slices.
- A judge can see the DreamDEX/Event Contract identity, source freshness, and contract-safe data boundary within one screen.

## 12. Design handoff checklist

Before frontend implementation begins:

1. Approve the route and page anatomy in Figma for desktop and mobile.
2. Resolve the API gaps for terms, `lockAt`, evidence, and canonical contract links.
3. Freeze fixture names and payloads for fresh, stale, locked, missing-terms, not-found, and upstream-error states.
4. Map each approved field to the OpenAPI response and generated client.
5. Write the Slice 1 implementation plan with no wallet/trade scope leakage.
6. Prepare the judge-facing demo script and evidence of the DreamDEX integration.

The implementation should start only after these gates pass. This keeps the Case File useful as a standalone product surface while preserving the contract-first discipline of Oraclelane.

## Related documents

- [Frontend slicing plan](24-frontend-slicing-plan.md)
- [Market Case File wireframe handoff](26-market-case-file-wireframe-handoff.md)
- [ADR-0009: dedicated detail contract](adr/0009-market-case-file-detail-contract.md)
