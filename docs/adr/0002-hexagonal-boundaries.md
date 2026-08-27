# ADR-0002: Isolate DreamDEX and infrastructure behind ports

**Date**: 2026-08-27  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

DreamDEX SDK/RPC APIs, LLM providers, and storage can change or fail independently. Protocol details must not leak into UI or domain policy.

## Decision

Use ports-and-adapters. Domain/application services depend on `MarketPort`, `ThesisPort`, `QuotePort`, `SettlementPort`, wallet-session, and repository ports; SDK/RPC/LLM/Postgres implementations remain adapters.

## Alternatives considered

### Direct SDK calls from controllers

- **Pros**: fewer files.
- **Cons**: hard to test, protocol coupling, unclear trust boundaries.
- **Why not**: makes settlement and safety behavior difficult to prove.

### Microservices

- **Pros**: independent scaling.
- **Cons**: network and operational complexity.
- **Why not**: unnecessary for the hackathon MVP.

## Consequences

Adapters require fixtures and mapping code, while domain tests stay deterministic and portable.
