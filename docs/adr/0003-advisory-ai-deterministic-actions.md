# ADR-0003: Keep AI advisory and actions deterministic

**Date**: 2026-08-27  
**Status**: accepted  
**Deciders**: Oraclelane team

## Context

AI can make a compelling explanation but is probabilistic and exposed to prompt injection. A prediction-market action also has financial consequences.

## Decision

The LLM returns only a validated `UP`, `DOWN`, or `NO_TRADE` thesis with evidence and expiry. Deterministic policies alone decide whether a user-visible trade preview is `ALLOW`, `WARN`, or `BLOCK`.

## Alternatives considered

### Autonomous trading agent

- **Pros**: more automation.
- **Cons**: custody/consent risk, harder demo proof, larger failure surface.
- **Why not**: outside MVP trust boundary.

### AI-free analytics

- **Pros**: simpler and deterministic.
- **Cons**: misses the product differentiation and onboarding value.
- **Why not**: AI is useful when constrained to explanation.

## Consequences

The product remains useful during LLM outages, but must invest in evidence validation and clear uncertainty copy.
