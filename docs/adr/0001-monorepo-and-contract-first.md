# ADR-0001: One implementation monorepo with contract-first boundaries

**Date**: 2026-08-27  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

The product will be built by a small team across UI/UX, frontend, backend, and chain integration. Separate repositories would add synchronization and release overhead before the product has stable boundaries.

## Decision

Keep planning in `blueprint`, technical decisions in `architecture`, and implementation in one future `app` monorepo. Use OpenAPI and shared event schemas as the authoritative boundary artifacts.

## Alternatives considered

### Polyrepo

- **Pros**: independent deploys and ownership.
- **Cons**: duplicated types, version drift, slower hackathon integration.
- **Why not**: team size and demo timeline favor one atomic change surface.

### Shared handwritten interfaces

- **Pros**: fast initially.
- **Cons**: runtime drift and undocumented errors.
- **Why not**: contract validation gives stronger integration proof.

## Consequences

The monorepo needs package boundaries and CI checks, but frontend can proceed with generated types and mocks while backend is incomplete.
