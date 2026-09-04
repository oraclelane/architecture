# 24. Frontend slicing plan

Slices are ordered vertical increments. Each slice can be reviewed and demoed independently using contract-valid fixtures. No slice begins by inventing API fields.

## Slice 0 — foundation + radar preview

**Scope:** app shell, shadcn theme, routing, typography, `ThemeProvider`, error/loading boundaries, generated API type pipeline, mock server, test harness, and the first `/radar` preview with responsive filters, refresh behavior, market cards, freshness/status badges, and skeleton/empty/stale/error states.

**Exit gate:** both themes render; keyboard navigation works; one market fixture renders through the typed fetcher; `market-open-fresh`, `market-stale`, and `market-locked` fixtures render correctly; stale/locked cards have no trade CTA; responsive filter/refresh interactions work; CI runs lint/typecheck/unit tests.

## Slice 1 — Market Case File

**Scope:** market case file route, terms, outcomes, price display, close/lock timing, evidence section, and refresh behavior.

**Contract:** `GET /markets/{marketId}`.

**Exit gate:** exact market question/terms are visible; IDs/amounts preserve string precision; unavailable data is explicit.

## Slice 2 — Thesis Panel

**Scope:** request action, loading, validated direction/confidence, drivers/risks, citations, expiry, `NO_TRADE`, and unavailable/rejected states.

**Contract:** `POST /theses`.

**Exit gate:** ADR-0010 and the amended OpenAPI are accepted; thesis cannot render an unknown citation or partial rejected output; market/evidence binding is preserved; expired thesis is visibly unusable for preview; `NO_TRADE` remains a complete valid state; idempotency and rate-limit behavior are tested.

## Slice 3 — Safety Gate and Trade Drawer

**Scope:** amount/outcome controls, deterministic checks, allow/warn/block presentation, quote expiry, and re-quote flow.

**Contract:** `POST /trade-previews`.

**Exit gate:** client cannot override `BLOCK`; changing amount or market version invalidates the preview; mobile uses an accessible `Drawer`.

## Slice 4 — Wallet and transaction lifecycle

**Scope:** connect/switch network, exact transaction review, user signing, rejection, pending, confirmed, and failed states.

**Contract:** `POST /transactions/observations` plus wallet adapter port.

**Exit gate:** no server signer; wallet rejection leaves no position; unknown receipt is reconciled by hash; transaction link is copyable and accessible.

## Slice 5 — Positions

**Scope:** `/positions`, detail route, filters, timeline, projection lag, and refresh after transaction confirmation.

**Contract:** `GET /positions`, `GET /positions/{positionId}`.

**Exit gate:** lifecycle states map one-to-one to documented enums; no client-only “success” state survives a reload.

## Slice 6 — Settlement and redeem

**Scope:** finalized/void summary, EIP-712 review, signature required, submitted, confirmed, expired, and manual fallback.

**Contract:** `POST /positions/{positionId}/redeem-intents`.

**Exit gate:** winning outcome is nullable for void; typed-data fields are displayed before signing; manual fallback remains available after automation failure.

## Slice 7 — Wallet session and health

**Scope:** connector selection, server-issued challenge, explicit signature step, session establishment, expiry and 401 re-authentication, and the operational health read. Protected surfaces stay empty until a session exists rather than rendering an unauthenticated shell.

**Contract:** `POST /auth/challenges`, `POST /auth/sessions`, `GET /health`. The bearer token from `POST /auth/sessions` is required by every operation the OpenAPI document marks `security: bearerAuth`.

**Exit gate:** the signable message is server-authored and signed verbatim, never composed on the client; a session issued for a different address or chain is refused before the credential is stored; the token is held in memory only, so a reload requires a new signature; a `401` on a protected read surfaces an explicit signature request instead of a silent retry; connecting a wallet without signing never counts as an authenticated session.

**Note:** this slice was specified after Slices 0-6 were written, because the session requirement in `11-security-architecture.md` was only exercised once the protected position and trade-preview reads were built. Slice 4 depends on it in practice: `POST /transactions/observations` is a protected endpoint.

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
