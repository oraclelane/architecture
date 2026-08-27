# ADR-0006: Versioned REST/OpenAPI boundary

**Date**: 2026-08-27  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

UI, API, and worker work will proceed in parallel. The frontend needs stable mocks and generated types before the provider is complete.

## Decision

Use task-oriented versioned REST under `/api/v1`, with `oraclelane.openapi.yaml` as the canonical contract, consistent envelopes, semantic statuses, cursor pagination, and explicit error codes.

## Alternatives considered

### GraphQL

- **Pros**: flexible reads.
- **Cons**: extra schema/runtime complexity for a small surface.
- **Why not**: REST resources map cleanly to the demo journeys.

### Backend-owned DTOs

- **Pros**: provider autonomy.
- **Cons**: consumer drift.
- **Why not**: incompatible with parallel frontend slicing.

## Consequences

Contract changes require review and regeneration, but integration failures become visible before deployment.
