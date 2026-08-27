# 4. Runtime data flows

## A. Discover → thesis → trade

```mermaid
sequenceDiagram
  participant U as User
  participant W as Web
  participant A as API
  participant D as DreamDEX adapter
  participant L as LLM
  participant C as Somnia contracts
  U->>W: Open market radar
  W->>A: GET /markets
  A->>D: Fetch + normalize live snapshot
  D-->>A: Snapshot + freshness
  A-->>W: Market cards
  U->>W: Open case file
  W->>A: POST /theses
  A->>D: Fetch evidence packet
  D-->>A: Sources + market state
  A->>L: Structured thesis request
  L-->>A: UP/DOWN/NO_TRADE + citations
  A-->>W: Validated thesis (expires)
  U->>W: Set amount + confirm
  W->>A: POST /trade-previews
  A->>D: Revalidate state + encode calldata
  A-->>W: Preview + safety decision
  U->>W: Sign transaction
  W->>C: User wallet submission
  C-->>W: Tx hash
  W->>A: POST /transactions/observations
```

## B. Finalize → redeem

```mermaid
sequenceDiagram
  participant S as Settlement worker
  participant C as DreamDEX settlement
  participant DB as Projection DB
  participant U as User wallet
  C-->>S: Settlement MarketFinalized event
  S->>C: Read payout vector + voided flag
  S->>DB: Record FINALIZED/VOID
  alt non-void and auto-redeem enabled
    S->>S: Derive argmax winning outcome
    S->>S: Create bounded EIP-712 payload
    S->>C: redeemFor(owner, nonce, deadline, signature,...)
    C-->>S: Redemption tx hash
    S->>DB: Record REDEEM_SUBMITTED
  else void or automation unavailable
    S->>DB: Record MANUAL_ACTION_REQUIRED
    U->>C: Manual redeem/claim via wallet
  end
```

Every transition is idempotent on `(chainId, marketKey, eventNonce)` or `(owner, nonce)`. A worker restart must not duplicate a redemption.
