# 16. Architecture sign-off

**Sign-off date:** 2026-08-27  
**Baseline:** `60ee75d` plus this gate update  
**Scope:** Oraclelane MVP for Somnia Shannon testnet

## Decision

The architecture is approved as the implementation baseline for UI/UX design and subsequent frontend/backend work. This approval does not authorize mainnet deployment, production custody, or scope expansion.

## Gate checklist

| Gate | Status | Evidence |
|---|---|---|
| Product scope mapped to technical boundaries | PASS | [system context](01-system-context.md), [implementation mapping](15-implementation-mapping.md) |
| Frontend/backend boundary is machine-readable | PASS | [OpenAPI contract](contracts/oraclelane.openapi.yaml) |
| DreamDEX protocol details isolated | PASS | [adapter design](07-dreamdex-adapter.md) |
| AI cannot initiate financial actions | PASS | [AI pipeline](09-ai-thesis-pipeline.md), ADR-0003 |
| Wallet remains user-controlled | PASS | [wallet execution](08-wallet-and-trade-execution.md), ADR-0004 |
| Settlement has automatic and manual paths | PASS | [settlement design](10-settlement-and-reactivity.md), ADR-0005 |
| Failure, observability, and rollback documented | PASS | [failure recovery](14-failure-and-recovery.md), [observability](13-observability.md) |
| Testnet environment explicitly bounded | PASS | [deployment](12-deployment-architecture.md) |
| No unresolved P0 architecture decision | PASS | ADR index and contract freeze below |

## Non-negotiable implementation constraints

1. Do not let SDK types leak into the web app or domain package.
2. Do not expose database rows directly as API responses.
3. Do not treat a cached/indexed market as tradeable when freshness is `STALE` or `UNKNOWN`.
4. Do not derive a settlement winner before finality, and never derive one for a void market.
5. Do not submit a transaction without an explicit user wallet action.
6. Do not ship a UI state that has no corresponding contract example or error behavior.

## Exit criteria for this gate

Architecture sign-off is complete when every new implementation PR links to the relevant document/ADR, passes contract validation, and records any intentional deviation before merge. A deviation that changes trust boundaries, money movement, or public API requires a new ADR and renewed sign-off.
