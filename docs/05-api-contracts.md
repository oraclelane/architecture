# 5. API contracts

The canonical HTTP contract is [`contracts/oraclelane.openapi.yaml`](contracts/oraclelane.openapi.yaml). Base path is `/api/v1`.

## Resources

| Endpoint | Consumer job | Notes |
|---|---|---|
| `GET /markets` | populate radar | cursor pagination; freshness included |
| `GET /markets/{marketId}` | render case file | opaque DreamDEX ID |
| `POST /theses` | request evidence-backed explanation | rate-limited; expires |
| `POST /trade-previews` | show deterministic preview/gate | no chain write |
| `POST /transactions/observations` | attach wallet tx hash | idempotent |
| `GET /positions` | show lifecycle | projection, reconciled to chain |
| `GET /positions/{positionId}` | show settlement/redeem CTA | manual fallback included |
| `POST /positions/{positionId}/redeem-intents` | request bounded redeem payload | never signs |
| `GET /health` | operational check | no secrets |

## Response and errors

Success uses `{ "data": ..., "meta": ... }`; errors use `{ "error": { "code", "message", "details" } }`. Use semantic HTTP statuses: `400` malformed, `401` unauthenticated, `403` unauthorized, `404` missing, `409` state conflict, `422` business validation, `429` rate limit, `502/503` upstream degradation.

Error messages are safe for the browser; provider details go to structured server logs keyed by `requestId`.

## Compatibility

`/api/v1` is additive-only during the hackathon. Breaking changes require `/api/v2` or a migration note. Unknown response fields must be ignored by clients; clients must not depend on undocumented fields.

## Contract workflow

Change OpenAPI first, review consumer impact, regenerate types/fixtures, update provider serializers, then run contract tests and one end-to-end path. No handwritten frontend-only payload interfaces.
