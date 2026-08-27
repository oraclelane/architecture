# Oraclelane Architecture

Technical source of truth for building **Oraclelane**, a trust and settlement copilot for DreamDEX Event Contracts on Somnia.

This repository explains *how the system is built*. Product intent, UX rationale, scope, and judging strategy live in the [Oraclelane blueprint](https://github.com/oraclelane/blueprint).

## Non-goals

- No market creation or market-maker logic.
- No autonomous entry, pooled custody, or complex strategy engine.
- No architecture that allows an LLM to submit transactions or custody keys.

## Build sequence

1. Freeze the boundary contracts in `docs/contracts/`.
2. Produce UI/UX designs from the journeys and states in the blueprint.
3. Slice the frontend against generated types and contract-valid mocks.
4. Implement backend, DreamDEX adapter, and settlement worker behind the contracts.
5. Integrate one vertical slice at a time; finish with testnet proof and demo hardening.

## Document map

| Document | Purpose |
|---|---|
| [System context](docs/01-system-context.md) | Actors, trust boundaries, and external systems |
| [Container architecture](docs/02-container-architecture.md) | Deployable runtime units and responsibilities |
| [Component architecture](docs/03-component-architecture.md) | Internal ports, adapters, and dependency rules |
| [Runtime flows](docs/04-runtime-data-flow.md) | Sequence diagrams for the critical user journeys |
| [API contracts](docs/05-api-contracts.md) | Versioned HTTP contract and error semantics |
| [Domain model](docs/06-domain-model.md) | Canonical entities, state machines, and invariants |
| [DreamDEX adapter](docs/07-dreamdex-adapter.md) | SDK/RPC/indexing boundaries and protocol facts |
| [Wallet and execution](docs/08-wallet-and-trade-execution.md) | Non-custodial signing and transaction lifecycle |
| [AI thesis pipeline](docs/09-ai-thesis-pipeline.md) | Evidence, model output, and deterministic safety gate |
| [Settlement and reactivity](docs/10-settlement-and-reactivity.md) | Redemption, EIP-712 authorization, and fallback |
| [Security architecture](docs/11-security-architecture.md) | Threat controls and abuse limits |
| [Deployment architecture](docs/12-deployment-architecture.md) | Testnet deployment topology and release provenance |
| [Observability](docs/13-observability.md) | Metrics, logs, traces, and judge-visible proof |
| [Failure recovery](docs/14-failure-and-recovery.md) | Failure matrix, retries, and operator runbook |
| [Implementation mapping](docs/15-implementation-mapping.md) | Architecture-to-code backlog and done criteria |
| [Architecture sign-off](docs/16-architecture-signoff.md) | Decision closure and implementation gate |
| [Contract freeze](docs/17-contract-freeze.md) | UI/backend boundary freeze and change protocol |
| [UI/UX design brief](docs/18-ui-ux-design-brief.md) | Design inputs, states, and handoff requirements |
| [ADR index](docs/adr/README.md) | Decisions and rejected alternatives |

## Contract-first rule

`docs/contracts/oraclelane.openapi.yaml` is the authoritative HTTP boundary. Event payloads and shared enums are defined in [domain model](docs/06-domain-model.md). Frontend mocks, generated types, and backend serializers must validate against these artifacts before integration.

## Status

Architecture baseline accepted for implementation planning. Runtime code, deployed addresses, and production credentials are intentionally absent.
