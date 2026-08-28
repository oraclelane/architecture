# 5. API contracts

The canonical HTTP contract is [`contracts/oraclelane.openapi.yaml`](contracts/oraclelane.openapi.yaml). Base path is `/api/v1`.

## Resources

| Endpoint | Consumer job | Notes |
|---|---|---|
| `POST /auth/challenges` | start wallet session | server-issued nonce/message; no auth required |
| `POST /auth/sessions` | establish owner session | verifies wallet signature; returns expiring bearer token |
| `GET /markets` | populate radar | cursor pagination; freshness included |
| `GET /markets/{marketId}` | render case file | opaque DreamDEX ID |
| `POST /theses` | request evidence-backed explanation | rate-limited; expires |
| `POST /trade-previews` | show deterministic preview/gate | no chain write |
| `POST /transactions/observations` | attach wallet tx hash | idempotent |
| `GET /positions` | show lifecycle | projection, reconciled to chain |
| `GET /positions/{positionId}` | show settlement/redeem CTA | owner session required; manual fallback included |
| `POST /positions/{positionId}/redeem-intents` | request bounded redeem payload | never signs |
| `POST /redeem-intents/{intentId}/signature` | submit owner EIP-712 signature | verifies and queues bounded operator submission |
| `GET /health` | operational check | no secrets |

## Response and errors

Success uses `{ "data": ..., "meta": ... }`; errors use `{ "error": { "code", "message", "details" } }`. Use semantic HTTP statuses: `400` malformed, `401` unauthenticated, `403` unauthorized, `404` missing, `409` state conflict, `422` business validation, `429` rate limit, `502/503` upstream degradation.

Error messages are safe for the browser; provider details go to structured server logs keyed by `requestId`.

## Compatibility

`/api/v1` is additive-only during the hackathon. Breaking changes require `/api/v2` or a migration note. Unknown response fields must be ignored by clients; clients must not depend on undocumented fields.

## Wallet session semantics

The client requests a short-lived challenge for a checksummed address and chain, signs the server-provided message in the wallet, and exchanges it for an opaque bearer session. The API derives the owner from the verified session—not from a trusted request body—and rejects a path/body address mismatch. Challenges and sessions are single-use/expiring and are never treated as trade authorization by themselves.

## Redeem signature semantics

`POST /positions/{positionId}/redeem-intents` creates a bounded, expiring EIP-712 payload. The browser displays it and calls wagmi/viem `signTypedData`. The browser then sends the signature to `POST /redeem-intents/{intentId}/signature`; the API validates domain, owner, nonce, deadline, and intent binding before placing it on a queue for the bounded operator/worker. The operator transaction hash and final status are projected back to the position. A user signature is never logged or accepted for a different position/intent.

## Contract workflow

Change OpenAPI first, review consumer impact, regenerate types/fixtures, update provider serializers, then run contract tests and one end-to-end path. No handwritten frontend-only payload interfaces.
