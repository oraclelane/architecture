# ADR-0010: Bind every thesis to a market snapshot and evidence packet

**Date**: 2026-08-30
**Status**: proposed
**Deciders**: Oraclelane team

## Context

The baseline `POST /theses` contract identified a market and returned direction, confidence, summary, evidence, and expiry. It did not bind the artifact to an opaque market version or evidence hash, did not include drivers/risks promised by the AI pipeline, and did not prevent clients from issuing duplicate expensive requests. It also could not distinguish a temporary provider outage from a candidate rejected for invalid schema or citations.

Without those boundaries, a valid-looking thesis could be shown beside a different market observation, unknown citations could be difficult to audit, and duplicate clicks could generate unnecessary model cost.

## Decision

`POST /api/v1/theses` remains a task-oriented public endpoint for the hackathon MVP, with these constraints:

1. The body contains only `marketId`, allowlisted `chainId`, and bounded `maxAgeSeconds`; additional properties are rejected.
2. The browser must send an `Idempotency-Key`. A key is bound to the normalized request and cannot be reused with a different body.
3. The server reloads the market and evidence. Clients cannot submit prompts, evidence, model names, direction, confidence, or expiry.
4. Every successful artifact includes `marketVersion`, `evidenceHash`, `promptVersion`, `modelVersion`, generation/expiry timestamps, drivers, risks, and allowlisted evidence metadata. A newly generated artifact returns `201`; a still-valid idempotent/cache reuse returns `200`.
5. `NO_TRADE` is a complete successful artifact with the same validation and evidence requirements as `UP` or `DOWN`.
6. Candidate output with invalid schema, uncertainty, expiry, or citations returns `502 THESIS_REJECTED`; temporary model/evidence degradation returns `503`.
7. The model remains advisory and has no wallet, trade, signing, database mutation, or contract-write tools.

## Alternatives considered

### Accept market facts, evidence, or a prompt from the client

- **Pros**: flexible experimentation and less server retrieval work.
- **Cons**: larger prompt-injection and SSRF surface, untrusted snapshot binding, difficult citation proof.
- **Why not**: violates the server-owned evidence and trust boundary.

### Return an unbound thesis keyed only by market ID

- **Pros**: smaller response and simpler cache.
- **Cons**: the same market ID can have newer prices/status/evidence; stale advice may appear current.
- **Why not**: insufficient for expiry and later trade-preview safety.

### Generate automatically when the Case File loads

- **Pros**: faster apparent UX.
- **Cons**: unnecessary provider cost, rate-limit abuse, hidden AI action without explicit intent.
- **Why not**: the thesis must be an explicit user request.

### Treat invalid model output as a successful `NO_TRADE`

- **Pros**: fewer visible failures.
- **Cons**: fabricates a decision and hides validation failures.
- **Why not**: `NO_TRADE` must be a validated model result; rejection remains explicit.

## Consequences

- Frontend and backend gain a reproducible artifact identity and deterministic expiry boundary.
- The API and fixtures become larger, but no handwritten frontend thesis type is permitted.
- Model/evidence failures remain non-fatal to the Case File.
- Idempotency storage and rate limiting are required before exposing the endpoint.
- Later trade previews may reference a thesis ID, but still must recompute all deterministic market and safety checks.
