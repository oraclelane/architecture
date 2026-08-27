# 11. Security architecture

## Threat controls

| Threat | Control | Evidence |
|---|---|---|
| LLM prompt injection | allowlisted sources, data delimiters, no tools, output validator | adversarial fixture tests |
| Wrong-chain transaction | chain ID + address allowlist + wallet switch check | preview contract test |
| Slippage/price movement | short expiry, re-read before encoding, max slippage gate | stale quote test |
| Replay redeem | owner/operator nonce, deadline, idempotency key | duplicate intent test |
| Custody compromise | no server signer/private key; browser-only wallet signing | architecture review |
| RPC/indexer poisoning | cross-check critical state with chain; source freshness | adapter tests |
| API abuse/cost blowup | wallet/IP rate limits, thesis quotas, request size limits | load test |
| XSS from evidence | render text safely, sanitize URLs, no raw HTML | frontend security test |
| Data leakage | redact signatures, tokens, prompts; encrypted secrets | log inspection |

## Authentication and authorization

Wallet sessions use a nonce challenge and signature verification; session ownership is tied to a checksummed address. Read endpoints can be public with rate limits. Position and redeem-intent endpoints require the owner session and verify the address matches the path/body.

## Secrets

LLM/API credentials live in the deployment secret manager. `.env` files are never committed. CI scans for secrets and fails before release.

## Auditability

Append-only audit records capture actor, action, market/position ID, decision, request ID, release SHA, and resulting tx hash. Sensitive payloads are hashed or redacted.
