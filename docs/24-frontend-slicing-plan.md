# 24. Frontend slicing plan

Slices are ordered vertical increments. Each slice can be reviewed and demoed independently using contract-valid fixtures. No slice begins by inventing API fields.

## Slice 0 — foundation

**Scope:** app shell, shadcn theme, routing, typography, `ThemeProvider`, error/loading boundaries, generated API type pipeline, mock server, and test harness.

**Exit gate:** both themes render; keyboard navigation works; one fixture renders through the typed fetcher; CI runs lint/typecheck/unit tests.

## Slice 1 — Market Radar

**Scope:** `/radar`, filters, market cards, freshness/status badges, skeleton/empty/stale/error states, and navigation to a case file.

**Contract:** `GET /markets`.

**Exit gate:** `market-open-fresh`, `market-stale`, and `market-locked` fixtures render correctly; stale/locked cards have no trade CTA.

## Slice 2 — Market Case File

**Scope:** terms, outcomes, price display, close/lock timing, evidence section, and refresh behavior.

**Contract:** `GET /markets/{marketId}`.

**Exit gate:** exact market question/terms are visible; IDs/amounts preserve string precision; unavailable data is explicit.

## Slice 3 — Thesis Panel

**Scope:** request action, loading, validated direction/confidence, drivers/risks, citations, expiry, `NO_TRADE`, and unavailable/rejected states.

**Contract:** `POST /theses`.

**Exit gate:** thesis cannot render an unknown citation; expired thesis is visibly unusable for preview; no-trade remains a valid state.

## Slice 4 — Safety Gate and Trade Drawer

**Scope:** amount/outcome controls, deterministic checks, allow/warn/block presentation, quote expiry, and re-quote flow.

**Contract:** `POST /trade-previews`.

**Exit gate:** client cannot override `BLOCK`; changing amount or market version invalidates the preview; mobile uses an accessible `Drawer`.

## Slice 5 — Wallet and transaction lifecycle

**Scope:** connect/switch network, exact transaction review, user signing, rejection, pending, confirmed, and failed states.

**Contract:** `POST /transactions/observations` plus wallet adapter port.

**Exit gate:** no server signer; wallet rejection leaves no position; unknown receipt is reconciled by hash; transaction link is copyable and accessible.

## Slice 6 — Positions

**Scope:** `/positions`, detail route, filters, timeline, projection lag, and refresh after transaction confirmation.

**Contract:** `GET /positions`, `GET /positions/{positionId}`.

**Exit gate:** lifecycle states map one-to-one to documented enums; no client-only “success” state survives a reload.

## Slice 7 — Settlement and redeem

**Scope:** finalized/void summary, EIP-712 review, signature required, submitted, confirmed, expired, and manual fallback.

**Contract:** `POST /positions/{positionId}/redeem-intents`.

**Exit gate:** winning outcome is nullable for void; typed-data fields are displayed before signing; manual fallback remains available after automation failure.

## Slice review checklist

Before a slice is marked complete:

- OpenAPI request/response and error paths are covered by fixtures;
- shadcn components use documented composition and semantic tokens;
- desktop/mobile and all required states are reviewed against Figma;
- keyboard, screen reader labels, and reduced-motion behavior are tested;
- no undocumented fields or client-side financial calculations were added;
- unit, integration, and critical E2E checks pass;
- the slice has a reproducible screenshot/trace for the eventual hackathon demo.

## Dependency rule

Slices may be built with mocks, but integration begins as soon as a provider path is available. Backend completion is not a prerequisite for reviewing frontend behavior, and frontend completion is not a reason to weaken the contract.
