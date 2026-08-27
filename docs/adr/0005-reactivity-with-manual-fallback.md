# ADR-0005: Reactivity with an explicit manual settlement fallback

**Date**: 2026-08-27  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

Reactive settlement is a key ecosystem demonstration, but runtime availability, finality, and operator submission can fail.

## Decision

Implement reactivity as an idempotent worker optimization around authoritative settlement events. Always expose a manual redemption path with clear state and transaction context.

## Alternatives considered

### Manual-only redemption

- **Pros**: minimal infrastructure.
- **Cons**: misses the differentiating automation proof.
- **Why not**: insufficient ecosystem impact narrative.

### Unbounded autonomous operator

- **Pros**: maximal automation.
- **Cons**: replay and permission risk.
- **Why not**: violates bounded authorization principle.

## Consequences

The worker needs event deduplication, dead-letter handling, and operational metrics, while users are never trapped by automation failure.
