# ADR-0011: Use Fastify, Drizzle and Postgres for the API runtime

**Date**: 2026-09-04
**Status**: accepted
**Deciders**: Oraclelane team

## Context

`02-container-architecture.md` names an API/BFF container, a worker container and a
managed Postgres, and ADR-0002 requires that every external dependency arrive with an ADR
and a failure note. The architecture documents deliberately stop at "task-oriented REST
API in TypeScript": no HTTP framework or persistence library had been chosen when the
frontend slices were built against fixtures.

Three obligations shape the choice, and all three are cross-cutting rather than
per-endpoint:

- every response carries a `requestId`, and errors must be safe for the browser while the
  provider detail goes to a structured log keyed by that id (`05-api-contracts.md`);
- thesis and market endpoints are rate limited, and origins are allowlisted
  (`11-security-architecture.md`);
- amounts and prices are exact strings, so nothing may round-trip through a float
  (`06-domain-model.md`, and the `Outcome.price` description in the contract).

## Decision

Build `apps/api` on **Fastify 5**, persist through **Drizzle ORM** over **Postgres**, and
keep the domain in a framework-free `packages/domain`.

- **Fastify** supplies request ids, a single error handler, CORS and rate limiting as
  first-class plugins, so the three cross-cutting obligations are configured once at the
  application root rather than repeated per route.
- **Drizzle** keeps the schema in TypeScript and emits plain SQL migrations that can be
  read and amended by hand. Migration `0001` uses that to make `audit_events` append-only
  with a trigger, which the append-only audit requirement needs enforced in the database
  rather than by convention.
- **Postgres columns for money-shaped values are `text`.** Base-unit amounts and decimal
  prices are stored exactly as the contract carries them; `numeric` would be defensible
  but `double precision` never is, and `text` removes the question.
- `packages/contracts` generates server-side types from the same frozen OpenAPI file the
  web app generates from, so a field means the same thing on both sides of the wire.

## Alternatives considered

### Next.js route handlers inside `apps/web`

- **Pros**: one deployment, no new container, shared types with no package boundary.
- **Cons**: contradicts the container split in `02-container-architecture.md`, and the
  settlement worker still needs its own long-lived process — the separation is deferred,
  not avoided. A request-scoped runtime is also a poor host for event subscriptions.
- **Why not**: it trades an architectural boundary for a saving that expires at E6.

### Hono

- **Pros**: smaller and portable across runtimes; Zod validation built in.
- **Cons**: the operational plugins this API needs on day one — rate limiting with a
  Retry-After contract, structured request logging — would be hand-written.
- **Why not**: no runtime portability requirement exists to pay for that work.

### Prisma

- **Pros**: mature, readable schema DSL, strong migration tooling.
- **Cons**: heavier client generation step, and the raw SQL needed for projection
  reconciliation and for an append-only trigger sits awkwardly outside its model.
- **Why not**: this API's hard parts are SQL-shaped, not model-shaped.

## Consequences

- The repository grows two package boundaries (`packages/domain`, `packages/contracts`)
  and a second application. Root `lint`, `typecheck`, `test` and `test:coverage` now run
  across every workspace, and CI additionally asserts that both generated clients still
  match the contract.
- Fastify plugin behaviour becomes load-bearing: `@fastify/rate-limit` throws whatever its
  `errorResponseBuilder` returns, so it returns an `ApiError` and the refusal is rendered
  by the same error handler as everything else. A future plugin upgrade must keep that
  path covered by the application tests.
- **Failure and fallback.** If Postgres is unavailable the API stays up and `GET /health`
  reports `degraded` naming the failed probe, rather than the process refusing to start;
  `14-failure-and-recovery.md` requires a degraded API with projections rebuildable from
  chain truth. If the chosen runtime ever has to change, the domain package moves
  unaltered — it imports no framework, driver or SDK.
