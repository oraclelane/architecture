# 23. Frontend implementation specification

This document translates the approved UX handoff into a future `apps/web` implementation. It is a plan and contract, not application source code.

## Route map

| Route | Rendering mode | Primary data | Primary action |
|---|---|---|---|
| `/radar` | server shell + client filters | `GET /markets` | open case file |
| `/markets/[chainId]/[marketId]` | server snapshot + client thesis | `GET /markets/{marketId}` | build thesis / review trade |
| `/positions` | server list + client refresh | `GET /positions` | inspect position |
| `/positions/[positionId]` | server detail + client wallet states | `GET /positions/{positionId}` | redeem/manual action |
| `/help/trust` | static/server | none | explain trust model |

Market and position IDs remain URL-safe opaque strings. Wallet signature payloads, private data, and raw prompts never enter URLs.

### Route decisions

- `/radar` is the canonical market discovery route; `/markets/[chainId]/[marketId]` is the canonical detail route.
- Landing page and settings are deferred from MVP. `/` may redirect to `/radar` but is not a separate product surface.
- `/help/trust` is the only supporting route required for the hackathon demo.

## Application shell

```text
<AppProviders>
  <ThemeProvider>
    <WagmiProvider>
      <QueryProvider>
        <RainbowKitProvider>
          <AppShell>
            <GlobalHeader />
            <NetworkBanner />
            <PageContainer />
            <ActivityRail />
            <Toaster />
          </AppShell>
        </RainbowKitProvider>
      </QueryProvider>
    </WagmiProvider>
  </ThemeProvider>
</AppProviders>
```

The shell owns theme, wallet connection, query cache, and global announcements. Feature pages own feature state; no global store should mirror the entire API cache.

## Wallet stack responsibilities

| Library | Responsibility | Boundary rule |
|---|---|---|
| RainbowKit | wallet connect modal, connector discovery, account/network UX | presentation only; no trade policy |
| wagmi | React hooks, wallet/account/chain state, transaction lifecycle, query integration | client boundary; expose typed actions to feature components |
| viem | chain definitions, public/wallet clients, ABI calldata, EIP-712 typed-data encoding, receipt reads | protocol boundary; preserve bigint/string precision and verified chain config |

`WagmiProvider` and `QueryClientProvider` are configured beneath the app theme and above feature routes; `RainbowKitProvider` wraps the wallet UI. The exact provider nesting may follow the selected RainbowKit/wagmi version, but the responsibilities must remain separated.

Use viem's chain configuration for Shannon/Elwood/mainnet only after the environment allowlist is verified. UI labels come from the active chain config, never from user input. Do not mix a second ethers/web3 client into the app.

## Component hierarchy

```text
features/markets
├── MarketRadarPage
│   ├── MarketFilters (ToggleGroup, Select)
│   ├── MarketList
│   │   └── MarketCard
│   └── MarketListState (Skeleton | Empty | Alert)
├── MarketCaseFilePage
│   ├── MarketFacts
│   ├── OutcomeSelector (ToggleGroup)
│   ├── EvidenceList
│   ├── ThesisPanel
│   └── TradeDrawer (Sheet/Drawer)
└── SafetyGate

features/positions
├── PositionsPage
│   ├── PositionFilters
│   ├── PositionList
│   │   └── PositionCard
│   └── PositionListState
└── PositionDetailPage
    ├── PositionTimeline
    ├── SettlementSummary
    └── RedeemAction
```

Feature components compose shadcn primitives; domain decisions stay in pure shared policies and are not recalculated from display labels.

## Server/client boundary

### Server-rendered responsibilities

- initial market/position fetch;
- SEO-safe market question and terms;
- serialization of contract-valid DTOs only;
- cache and revalidation policy for read data;
- no wallet provider or browser-only APIs.

### Client responsibilities

- filters, tabs, selected outcome, amount input;
- wallet connection, chain switching, signature prompts;
- thesis request trigger and expiry countdown;
- trade drawer and transaction observation;
- accessible announcements and optimistic UI limited to non-financial presentation.

Any component using state, effects, event handlers, or wallet/browser APIs is explicitly client-side. Keep client props small; pass IDs and required fields, not entire database-shaped objects.

## Data fetching and cache policy

- Use one typed fetcher generated from OpenAPI.
- Query keys include `chainId`, resource ID, filters, and contract/version where relevant.
- Market radar may use a short stale-while-revalidate window for rendering; a trade preview always triggers a fresh server validation.
- Thesis responses are cached only by `marketId + snapshotVersion + evidenceHash` and expire with the thesis.
- Position detail revalidates after a submitted transaction and on explicit user refresh.
- Abort in-flight requests when a route changes; avoid duplicate global listeners.
- Recommended query keys are `['markets', chainId, filters]`, `['market', chainId, marketId]`, and `['positions', owner, chainId]`.
- Retry only idempotent reads with bounded backoff. Never automatically retry wallet rejection, `422`, or `409` responses.

## Local state ownership

| State | Owner | Persistence |
|---|---|---|
| theme | app shell | user preference |
| wallet address/chain | wallet provider | provider/session only |
| filters | route/page | URL query or local state |
| selected outcome/amount | trade drawer | memory only until preview |
| preview expiry | trade drawer | memory; discard on expiry |
| thesis request status | case file | query cache, expiry-bound |
| tx hash/status | position feature | API projection; reconcile by hash |
| toast/announcement | app shell | ephemeral |

Never persist private keys, signatures, raw prompts, or an unconfirmed financial outcome in local storage.

## Presentation state mapper

Components consume a finite presentation state derived from canonical API enums; they do not invent a parallel financial state machine.

```text
loading | empty | fresh | stale | unavailable | blocked |
pending | success | failed | manual
```

The mapper must use an exhaustive switch and fail closed to `unavailable` for an unknown API value. `amount` remains a decimal string through the UI; formatting utilities may add separators but must never convert it to a JavaScript `number`.

## Wallet boundary

```text
web -> POST /trade-previews -> display exact fields -> wallet.request()
web -> POST /transactions/observations(txHash) -> projection polling
web -> POST /redeem-intents -> wallet.signTypedData() -> bounded submit path
```

The planned client APIs map to the stack as follows:

- `useConnect`, `useAccount`, and `useChainId` for connection and network state;
- `useSwitchChain` for an explicit wrong-chain recovery action;
- `useSendTransaction` (or the version-equivalent wagmi mutation) for a trade preview transaction;
- `useSignTypedData` for owner-approved redeem authorization;
- viem public-client reads/watches for receipt and finality display, with the API projection remaining the lifecycle authority.

Names are version-sensitive implementation details; the contract and trust boundary are stable. Pin compatible library versions together in the future app lockfile and test upgrades as one wallet-stack change.

The frontend may display and submit a wallet-created transaction. It cannot edit calldata, alter safety checks, or bypass a `BLOCK` result. Unknown transaction receipts are reconciled by hash before any retry.

## Contract blockers before implementation

The following items are intentionally recorded rather than guessed. Frontend slicing can proceed with mocks, but the `app` repository must not be considered integration-ready until each item is resolved in the OpenAPI contract and an ADR if the trust boundary changes.

| Blocker | Required resolution |
|---|---|
| Position detail path | Add and validate `GET /positions/{positionId}` with owner/chain authorization semantics, or remove the route dependency. |
| Signed redeem submission | Define whether the browser submits directly or sends the owner signature to a bounded operator. Add the required request/observation contract. |
| Owner authentication | Specify wallet nonce challenge/session transport, address matching, expiry, and replay protection for owner-scoped endpoints. |
| Contract route parity | Keep `/radar` and `/markets/...` as UI routes while API remains `/markets`; document any redirect or cache-key behavior. |

## Error and loading composition

Every route defines an error boundary and a route-level loading state. Use `Skeleton`, `Empty`, `Alert`, `Spinner`, `Progress`, and `sonner` according to the design system. Error copy is actionable and includes a support correlation ID where appropriate; provider stack traces never reach the browser.

## Performance guardrails

- Parallelize independent server reads.
- Dynamically load the chart bundle only when the case file requests it.
- Avoid barrel imports for large icon/chart packages.
- Memoize only expensive derived lists; derive simple booleans during render.
- Virtualize only if the market list exceeds the agreed threshold; do not prematurely add complexity.
- Keep wallet and analytics code out of the initial radar bundle where possible.

## Frontend security checklist

- Validate all URL/query params with the same schemas as API requests.
- Render evidence titles/descriptions as text; sanitize and allowlist external URLs.
- Use semantic shadcn tokens; no user-controlled class names or raw HTML.
- Redact signatures and sensitive request data from client logs.
- Enforce origin/CSRF strategy for mutation requests and wallet session checks.
- Verify chain ID and contract target immediately before wallet submission.

## Test plan

- Unit: presentation mapper, amount formatting, URL parameter schemas, and state transitions.
- Component: every required fixture state, keyboard flow, focus restoration, and accessible status text.
- Integration: MSW handlers and generated OpenAPI types; validate exact envelopes and errors.
- Wallet adapter: wrong chain, user rejection, pending, replacement, revert, and unknown receipt.
- E2E: Radar → Case File → Preview `BLOCK`/`ALLOW` → wallet reject/success → Position → finalized/void → redeem/manual fallback.
- Accessibility: axe/contrast, 44px touch targets, reduced motion, and screen-reader labels.
