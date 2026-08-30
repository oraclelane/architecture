# Slice 2 Thesis Panel Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an explicit, evidence-backed Thesis Panel to the existing Market Case File so a user can request, inspect, expire, and safely recover from a validated `UP`, `DOWN`, or `NO_TRADE` advisory without adding wallet or trading authority.

**Architecture:** Keep the Case File route and server-fetched market facts intact. Add a generated-contract-backed `createThesis` client, a pure exhaustive presentation-state mapper, a request lifecycle controller, and a shadcn/ui-based panel inside the existing Case File surface. The client accepts only market identity and bounded cache age, preserves the last complete validated artifact, rejects response identity mismatches, and treats expiry and every safe API failure as explicit UI states.

**Tech Stack:** Next.js 15 App Router, React 19, TypeScript 5.8, generated OpenAPI types, Zod 3, official shadcn/ui components built on Base UI, Vitest 3, Testing Library, MSW 2, Playwright.

**Spec:** `architecture/docs/27-thesis-panel-architecture.md`, `architecture/docs/adr/0010-bind-theses-to-market-and-evidence.md`, `architecture/docs/contracts/oraclelane.openapi.yaml`, `architecture/docs/20-design-system.md`, `architecture/docs/21-screen-handoff.md`, and `architecture/docs/22-ux-copy-and-states.md`.

## Global Constraints

- Do not add wallet connection, RainbowKit, wagmi, viem, trade preview, signing, redeem, positions, market creation, pooled custody, autonomous entry, or strategy execution in Slice 2.
- Do not add arbitrary prompts, client-submitted evidence, model selection, expiry selection, or provider data to the browser request.
- Use only types generated from `architecture/docs/contracts/oraclelane.openapi.yaml`; do not create handwritten thesis DTOs.
- Parse every successful HTTP response with a strict runtime schema before rendering. Unknown keys, invalid citations, invalid timestamps, or identity mismatches fail closed.
- Render model-controlled strings as React text only. Never use `dangerouslySetInnerHTML` for summaries, drivers, risks, or evidence.
- Treat `NO_TRADE` as a complete successful result with the same evidence and expiry structure as a directional thesis.
- Keep an expired artifact visible for audit, but mark it unusable for any future preview.
- Do not automatically generate or retry a thesis. Every initial request and recovery retry must be an explicit user action.
- Preserve valid Case File facts throughout thesis loading and failure states.
- Use existing official shadcn/ui primitives from `apps/web/src/components/ui`; do not introduce a parallel bespoke component library.
- Follow TDD for every task: establish RED, implement the smallest GREEN change, refactor, then run the focused test again.
- Maintain at least 80% line, function, branch, and statement coverage.
- Before each application commit, run the focused tests, inspect `git diff --check`, and scan the diff for secrets and unsafe HTML.
- The application repository is `/Users/mac/developer/oraclelane`. The architecture repository is the nested Git submodule `/Users/mac/developer/oraclelane/architecture`; commit them separately and update the parent submodule pointer only after the architecture commit exists.

---

## Delivered prerequisite: contract fixtures

The implementation starts from these already-created, generated-type-backed fixtures:

- `apps/web/src/lib/fixtures/theses.ts`
- `apps/web/src/lib/fixtures/theses.test.ts`
- `apps/web/src/mocks/handlers.ts`
- `apps/web/src/mocks/handlers.test.ts`

They cover `thesis-valid-up`, `thesis-valid-no-trade`, `thesis-expired`, `thesis-market-conflict`, `thesis-rate-limited`, `thesis-rejected`, and `thesis-unavailable`. The primary success fixture is bound to the existing Radar market `fixture-btc-18utc` so the critical demo path remains coherent.

Preflight:

```bash
cd /Users/mac/developer/oraclelane
pnpm generate:api
pnpm --filter @oraclelane/web exec vitest run src/lib/fixtures/theses.test.ts src/mocks/handlers.test.ts
git diff --check
```

Expected: generated types remain current, 27 fixture/handler tests pass, and no whitespace error is reported.

## Planned file map

**Modify**

- `apps/web/src/lib/api/types.ts`
- `apps/web/src/lib/api/client.ts`
- `apps/web/src/lib/api/client.test.ts`
- `apps/web/src/features/case-file/case-file-sections.tsx`
- `apps/web/src/features/case-file/case-file-surface.tsx`
- `apps/web/src/features/case-file/case-file-surface.test.tsx`
- `apps/web/src/features/case-file/case-file.css`
- `apps/web/package.json`
- `pnpm-lock.yaml`
- `.github/workflows/ci.yml`
- `README.md`
- `apps/web/README.md`

**Create**

- `apps/web/src/features/case-file/thesis-panel-model.ts`
- `apps/web/src/features/case-file/thesis-panel-model.test.ts`
- `apps/web/src/features/case-file/thesis-panel.tsx`
- `apps/web/src/features/case-file/thesis-panel.test.tsx`
- `apps/web/src/features/case-file/thesis-panel-controller.tsx`
- `apps/web/src/features/case-file/thesis-panel-controller.test.tsx`
- `apps/web/playwright.config.ts`
- `apps/web/e2e/thesis-panel.spec.ts`

---

## Task 1: Implement the strict thesis API boundary

**Files:**

- Modify: `apps/web/src/lib/api/types.ts`
- Modify: `apps/web/src/lib/api/client.ts`
- Modify: `apps/web/src/lib/api/client.test.ts`
- Read only: `apps/web/src/lib/api/generated.ts`
- Read only: `apps/web/src/lib/fixtures/theses.ts`

- [ ] **Step 1: Write failing request and response boundary tests**

Extend `apps/web/src/lib/api/client.test.ts` with tests that prove:

```ts
const validRequest = { marketId: "fixture-btc-18utc", chainId: 50312, maxAgeSeconds: 300 } as const;

function validHttpOptions(fetcher: typeof fetch) {
  return {
    mode: "http" as const,
    baseUrl: "https://api.oraclelane.test/api/v1",
    fetcher,
    idempotencyKey: "fixture-idempotency-key-0001",
  };
}

function responseWithThesisPatch(patch: Partial<typeof thesisValidUpFixture.data>) {
  return new Response(JSON.stringify({
    ...thesisValidUpFixture,
    data: { ...thesisValidUpFixture.data, ...patch },
  }), { status: 201, headers: { "content-type": "application/json" } });
}

describe("createThesis", () => {
  it("posts only the contract request with an idempotency key", async () => {
    const fetcher = vi.fn(async (_input: RequestInfo | URL, init?: RequestInit) => {
      expect(init?.method).toBe("POST");
      expect(new Headers(init?.headers).get("Idempotency-Key")).toBe("fixture-idempotency-key-0001");
      expect(JSON.parse(String(init?.body))).toEqual({
        marketId: "fixture-btc-18utc",
        chainId: 50312,
        maxAgeSeconds: 300,
      });
      return new Response(JSON.stringify(thesisValidUpFixture), {
        status: 201,
        headers: { "content-type": "application/json" },
      });
    });

    await expect(createThesis(
      { marketId: "fixture-btc-18utc", chainId: 50312, maxAgeSeconds: 300 },
      { mode: "http", baseUrl: "https://api.oraclelane.test/api/v1", fetcher, idempotencyKey: "fixture-idempotency-key-0001" },
    )).resolves.toEqual(thesisValidUpFixture);
  });

  it("rejects unknown response keys and market identity mismatches", async () => {
    const extraFieldFetcher = async () => new Response(JSON.stringify({ ...thesisValidUpFixture, internalPrompt: "do not expose" }), { status: 201 });
    await expect(createThesis(validRequest, validHttpOptions(extraFieldFetcher))).rejects.toThrow();

    const mismatchFetcher = async () => new Response(JSON.stringify({
      ...thesisValidUpFixture,
      data: { ...thesisValidUpFixture.data, marketId: "another-market" },
    }), { status: 201 });
    await expect(createThesis(validRequest, validHttpOptions(mismatchFetcher))).rejects.toMatchObject({
      status: 502,
      code: "THESIS_IDENTITY_MISMATCH",
    });
  });

  it.each([
    ["malformed generatedAt", () => responseWithThesisPatch({ generatedAt: "not-a-date" })],
    ["non-HTTPS evidence URL", () => responseWithThesisPatch({ evidence: [{ ...thesisValidUpFixture.data.evidence[0], url: "http://example.test/source" }] })],
    ["invalid evidence hash", () => responseWithThesisPatch({ evidenceHash: "sha256:not-valid" })],
    ["empty drivers", () => responseWithThesisPatch({ drivers: [] })],
    ["more than three risks", () => responseWithThesisPatch({ risks: ["one", "two", "three", "four"] })],
    ["missing evidence", () => responseWithThesisPatch({ evidence: [] })],
  ])("fails closed for %s", async (_case, buildResponse) => {
    const fetcher = async () => buildResponse();

    await expect(createThesis(validRequest, validHttpOptions(fetcher))).rejects.toThrow();
  });

  it("preserves Retry-After on a safe rate-limit error", async () => {
    await expect(createThesis(
      { marketId: THESIS_FIXTURE_MARKET_IDS.rateLimited, chainId: 50312, maxAgeSeconds: 300 },
      { mode: "http", baseUrl: "https://api.oraclelane.test/api/v1", idempotencyKey: "fixture-idempotency-key-0002" },
    )).rejects.toMatchObject({ status: 429, code: "THESIS_RATE_LIMITED", retryAfterSeconds: 60 });
  });
});
```

- [ ] **Step 2: Run the API tests and verify RED**

```bash
cd /Users/mac/developer/oraclelane
pnpm --filter @oraclelane/web exec vitest run src/lib/api/client.test.ts
```

Expected: FAIL because `createThesis`, thesis runtime validation, and `retryAfterSeconds` do not exist.

- [ ] **Step 3: Extend the safe API error type**

In `apps/web/src/lib/api/types.ts`, extend `ApiClientError` without exposing response bodies:

```ts
export class ApiClientError extends Error {
  readonly status: number;
  readonly requestId?: string;
  readonly code?: string;
  readonly retryAfterSeconds?: number;

  constructor(message: string, options: {
    status: number;
    requestId?: string;
    code?: string;
    retryAfterSeconds?: number;
  }) {
    super(message);
    this.name = "ApiClientError";
    this.status = options.status;
    this.requestId = options.requestId;
    this.code = options.code;
    this.retryAfterSeconds = options.retryAfterSeconds;
  }
}
```

Keep the existing generated aliases for `ThesisRequest`, `Thesis`, `ThesisResponse`, `ThesisMeta`, and `Evidence`.

- [ ] **Step 4: Add strict Zod schemas and `createThesis`**

In `apps/web/src/lib/api/client.ts`, build strict schemas mirroring the accepted OpenAPI constraints. Use `.strict()` on every thesis object and error boundary. The success schema must include:

```ts
const evidenceSchema = z.object({
  sourceId: z.string().min(1).max(128),
  title: z.string().min(1).max(240),
  url: z.string().url().startsWith("https://"),
  publishedAt: z.string().datetime().nullable().optional(),
  retrievedAt: z.string().datetime(),
  relevance: z.enum(["PRIMARY", "SECONDARY", "MARKET_DATA"]),
}).strict();

const thesisResponseSchema = z.object({
  data: z.object({
    thesisId: z.string().min(1),
    marketId: z.string().min(1),
    chainId: z.union([z.literal(50312), z.literal(50313), z.literal(5031)]),
    marketVersion: z.string().min(1),
    direction: z.enum(["UP", "DOWN", "NO_TRADE"]),
    confidenceBand: z.enum(["LOW", "MEDIUM", "HIGH"]),
    summary: z.string().min(1).max(1200),
    drivers: z.array(z.string().min(1).max(280)).min(1).max(3),
    risks: z.array(z.string().min(1).max(280)).min(1).max(3),
    evidence: z.array(evidenceSchema).min(1).max(8),
    evidenceHash: z.string().regex(/^sha256:[a-f0-9]{64}$/),
    generatedAt: z.string().datetime(),
    expiresAt: z.string().datetime(),
    modelVersion: z.string().min(1).max(128),
    promptVersion: z.string().min(1).max(128),
  }).strict(),
  meta: z.object({
    requestId: z.string().min(1),
    cacheStatus: z.enum(["HIT", "MISS"]),
  }).strict(),
}).strict().superRefine((value, context) => {
  if (Date.parse(value.data.expiresAt) <= Date.parse(value.data.generatedAt)) {
    context.addIssue({ code: z.ZodIssueCode.custom, message: "Thesis expiry must follow generation" });
  }
});
```

Add `idempotencyKey` to a thesis-specific options type and implement:

```ts
export type ThesisFetchOptions = FetchOptions & Readonly<{ idempotencyKey: string }>;

export async function createThesis(
  request: ThesisRequest,
  options: ThesisFetchOptions,
): Promise<ThesisResponse> {
  const parsedRequest = thesisRequestSchema.parse(request);
  const idempotencyKey = idempotencyKeySchema.parse(options.idempotencyKey);

  if ((options.mode ?? getApiMode()) === "fixture") {
    return resolveThesisFixture(parsedRequest);
  }

  const baseUrl = getBaseUrl(options.baseUrl);
  assertServerHttpBaseUrl(baseUrl);
  const response = await (options.fetcher ?? fetch)(buildRequestUrl("/theses", baseUrl), {
    method: "POST",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      "Idempotency-Key": idempotencyKey,
    },
    body: JSON.stringify(parsedRequest),
    signal: options.signal,
    cache: "no-store",
  });

  await assertSuccessfulResponse(response, "Thesis");
  const parsedResponse = thesisResponseSchema.parse(await response.json()) as ThesisResponse;
  if (parsedResponse.data.marketId !== parsedRequest.marketId || parsedResponse.data.chainId !== parsedRequest.chainId) {
    throw new ApiClientError("Thesis response market identity mismatch", {
      status: 502,
      code: "THESIS_IDENTITY_MISMATCH",
    });
  }
  return parsedResponse;
}
```

`resolveThesisFixture` must use `thesisSuccessFixtures` and `thesisErrorFixtures`, clone returned arrays/objects, and throw `ApiClientError` for error fixtures. It must not mutate exported fixtures.

- [ ] **Step 5: Make shared error parsing preserve safe headers**

Change the internal success assertion to accept a resource label and parse `Retry-After` only as a non-negative integer:

```ts
function parseRetryAfterSeconds(response: Response) {
  const value = response.headers.get("Retry-After");
  if (!value || !/^\d+$/.test(value)) return undefined;
  return Number(value);
}
```

Never return provider text, stack traces, prompt content, or invalid `Retry-After` values.

- [ ] **Step 6: Run focused and existing API tests**

```bash
pnpm --filter @oraclelane/web exec vitest run src/lib/api/client.test.ts src/lib/fixtures/theses.test.ts src/mocks/handlers.test.ts
pnpm typecheck
git diff --check
```

Expected: all focused tests pass, typecheck passes, and market client behavior remains unchanged.

- [ ] **Step 7: Commit Task 1**

```bash
git add apps/web/src/lib/api/types.ts apps/web/src/lib/api/client.ts apps/web/src/lib/api/client.test.ts
git commit -m "feat: add strict thesis API client"
```

---

## Task 2: Define the exhaustive Thesis Panel state model

**Files:**

- Create: `apps/web/src/features/case-file/thesis-panel-model.ts`
- Create: `apps/web/src/features/case-file/thesis-panel-model.test.ts`

- [ ] **Step 1: Write failing state-classification tests**

Cover all presentation states with an explicit clock:

```ts
describe("classifyThesisResponse", () => {
  it("classifies UP and DOWN as valid directional results", () => {
    expect(classifyThesisResponse(thesisValidUpFixture, Date.parse("2031-08-30T09:02:00Z"))).toMatchObject({
      kind: "valid_directional",
      response: thesisValidUpFixture,
    });
  });

  it("keeps NO_TRADE successful", () => {
    expect(classifyThesisResponse(thesisValidNoTradeFixture, Date.parse("2031-08-30T09:02:00Z"))).toMatchObject({
      kind: "valid_no_trade",
    });
  });

  it("expires exactly at expiresAt without deleting evidence", () => {
    const state = classifyThesisResponse(thesisExpiredFixture, Date.parse(thesisExpiredFixture.data.expiresAt));
    expect(state.kind).toBe("expired");
    if (state.kind === "expired") expect(state.response.data.evidence).toEqual(thesisExpiredFixture.data.evidence);
  });
});

describe("classifyThesisError", () => {
  it.each([
    [409, "market_conflict"],
    [429, "rate_limited"],
    [502, "rejected"],
    [503, "unavailable"],
    [0, "unavailable"],
  ])("maps %s to %s", (status, kind) => {
    expect(classifyThesisError(new ApiClientError("safe", { status }))).toMatchObject({ kind });
  });
});
```

Also test invalid timestamps fail to `unavailable`, `retryAfterSeconds` is retained only for `rate_limited`, and exhaustive rendering rejects unknown directions at compile time.

- [ ] **Step 2: Run the model test and verify RED**

```bash
pnpm --filter @oraclelane/web exec vitest run src/features/case-file/thesis-panel-model.test.ts
```

Expected: FAIL because the model module does not exist.

- [ ] **Step 3: Implement the discriminated union and pure classifiers**

Create `apps/web/src/features/case-file/thesis-panel-model.ts`:

```ts
import { ApiClientError, type ThesisResponse } from "@/lib/api/types";

export type ThesisPanelState =
  | Readonly<{ kind: "not_requested" }>
  | Readonly<{ kind: "loading"; previousResponse?: ThesisResponse }>
  | Readonly<{ kind: "valid_directional"; response: ThesisResponse }>
  | Readonly<{ kind: "valid_no_trade"; response: ThesisResponse }>
  | Readonly<{ kind: "expired"; response: ThesisResponse }>
  | Readonly<{ kind: "market_conflict"; message: string; previousResponse?: ThesisResponse }>
  | Readonly<{ kind: "rate_limited"; message: string; retryAfterSeconds?: number; previousResponse?: ThesisResponse }>
  | Readonly<{ kind: "rejected"; message: string; previousResponse?: ThesisResponse }>
  | Readonly<{ kind: "unavailable"; message: string; previousResponse?: ThesisResponse }>;

export function classifyThesisResponse(response: ThesisResponse, now = Date.now()): ThesisPanelState {
  const expiresAt = Date.parse(response.data.expiresAt);
  if (!Number.isFinite(expiresAt)) return { kind: "unavailable", message: "The thesis expiry is invalid." };
  if (now >= expiresAt) return { kind: "expired", response };
  if (response.data.direction === "NO_TRADE") return { kind: "valid_no_trade", response };
  return { kind: "valid_directional", response };
}

export function classifyThesisError(error: unknown, previousResponse?: ThesisResponse): ThesisPanelState {
  if (!(error instanceof ApiClientError)) {
    return { kind: "unavailable", message: "The thesis service is temporarily unavailable.", previousResponse };
  }
  if (error.status === 409) return { kind: "market_conflict", message: error.message, previousResponse };
  if (error.status === 429) return { kind: "rate_limited", message: error.message, retryAfterSeconds: error.retryAfterSeconds, previousResponse };
  if (error.status === 502) return { kind: "rejected", message: error.message, previousResponse };
  return { kind: "unavailable", message: error.message, previousResponse };
}
```

Do not add a generic `error` state. The union must preserve the recovery policy of each failure.

- [ ] **Step 4: Add deterministic time copy helpers**

Add pure helpers for exact UTC expiry and bounded remaining time. Return `Expired` at or below zero; do not display negative time or imply the browser extended the server expiry.

- [ ] **Step 5: Run focused tests and typecheck**

```bash
pnpm --filter @oraclelane/web exec vitest run src/features/case-file/thesis-panel-model.test.ts
pnpm typecheck
git diff --check
```

Expected: all state-model tests pass and TypeScript confirms the discriminated union.

- [ ] **Step 6: Commit Task 2**

```bash
git add apps/web/src/features/case-file/thesis-panel-model.ts apps/web/src/features/case-file/thesis-panel-model.test.ts
git commit -m "feat: model exhaustive thesis states"
```

---

## Task 3: Build the accessible shadcn/ui Thesis Panel

**Files:**

- Create: `apps/web/src/features/case-file/thesis-panel.tsx`
- Create: `apps/web/src/features/case-file/thesis-panel.test.tsx`
- Reuse: `apps/web/src/components/ui/alert.tsx`
- Reuse: `apps/web/src/components/ui/badge.tsx`
- Reuse: `apps/web/src/components/ui/button.tsx`
- Reuse: `apps/web/src/components/ui/card.tsx`
- Reuse: `apps/web/src/components/ui/separator.tsx`

- [ ] **Step 1: Write failing rendering tests for every state**

The test table must assert user-visible behavior, not component internals:

```ts
it.each([
  ["not_requested", "Build a thesis"],
  ["loading", "Building thesis…"],
  ["market_conflict", "Refresh market"],
  ["rate_limited", "Try again"],
  ["rejected", "Thesis not shown"],
  ["unavailable", "Try again"],
])("renders the %s recovery contract", (kind, expectedAction) => {
  render(<ThesisPanel state={stateFor(kind)} onRequest={onRequest} onRetry={onRetry} onRefreshMarket={onRefreshMarket} />);
  expect(screen.getByText(/Oraclelane thesis · advisory, not a guarantee/i)).toBeInTheDocument();
  expect(screen.getByRole("button", { name: expectedAction })).toBeInTheDocument();
});
```

Add specific tests proving:

- `UP` and `DOWN` render direction, confidence, summary, drivers, risks, all evidence links, generated time, expiry, model version, prompt version, and cache status;
- `NO_TRADE` uses a successful advisory card and the copy `Waiting is a valid outcome`;
- `expired` retains every citation but announces `Expired · cannot be used for a later preview`;
- successful and expired artifacts provide one explicit `Build another thesis` action; they never refresh themselves automatically;
- `rejected` and `unavailable` never render any summary, driver, risk, or citation from a partial payload;
- loading or failure after a previously validated request keeps that older artifact visible under the label `Previous validated thesis`; it must never mix old fields with the new partial response;
- loading uses `aria-busy="true"`, keeps one disabled button, and exposes a polite live status;
- evidence links are HTTPS and use `target="_blank" rel="noreferrer noopener"`;
- no button or link matches `/connect wallet|trade|redeem|sign/i`.

- [ ] **Step 2: Run the component test and verify RED**

```bash
pnpm --filter @oraclelane/web exec vitest run src/features/case-file/thesis-panel.test.tsx
```

Expected: FAIL because `ThesisPanel` does not exist.

- [ ] **Step 3: Implement the state shell with official shadcn/ui primitives**

Use this public component boundary:

```ts
type ThesisPanelProps = Readonly<{
  state: ThesisPanelState;
  eligible: boolean;
  eligibilityReason?: string;
  now: number;
  onRequest: () => void;
  onRetry: () => void;
  onRefreshMarket: () => void;
}>;
```

The outer structure is a `Card` with `CardHeader`, `CardContent`, and `CardFooter`. Use `Badge` for direction, confidence, cache state, and expiry; `Alert` for conflict/rejected/unavailable states; `Button` for explicit actions; and `Separator` between interpretation and provenance.

Use an exhaustive `switch` and a `never` guard:

```ts
function assertNever(value: never): never {
  throw new Error(`Unhandled Thesis Panel state: ${JSON.stringify(value)}`);
}
```

The thrown value is a developer guard and must not be rendered to users.

- [ ] **Step 4: Implement the artifact body without unsafe HTML**

Render strings directly:

```tsx
<p>{response.data.summary}</p>
<ul aria-label="Thesis drivers">
  {response.data.drivers.map((driver) => <li key={driver}>{driver}</li>)}
</ul>
<ul aria-label="Thesis risks">
  {response.data.risks.map((risk) => <li key={risk}>{risk}</li>)}
</ul>
```

Evidence keys must use `sourceId`; the strict API boundary already rejects malformed response shapes. Show `retrievedAt`, optional `publishedAt`, and `relevance` for each source.

- [ ] **Step 5: Run component, model, and UI primitive tests**

```bash
pnpm --filter @oraclelane/web exec vitest run \
  src/features/case-file/thesis-panel.test.tsx \
  src/features/case-file/thesis-panel-model.test.ts \
  src/components/ui/ui.test.tsx
pnpm typecheck
git diff --check
```

Expected: all tests pass and no new component primitive is introduced.

- [ ] **Step 6: Commit Task 3**

```bash
git add apps/web/src/features/case-file/thesis-panel.tsx apps/web/src/features/case-file/thesis-panel.test.tsx
git commit -m "feat: render accessible thesis panel"
```

---

## Task 4: Implement request lifecycle, idempotency, abort, and expiry

**Files:**

- Create: `apps/web/src/features/case-file/thesis-panel-controller.tsx`
- Create: `apps/web/src/features/case-file/thesis-panel-controller.test.tsx`
- Modify only if required by tests: `apps/web/src/features/case-file/thesis-panel-model.ts`

- [ ] **Step 1: Write failing controller tests**

Use fake timers and deferred promises to prove:

1. no request occurs on mount;
2. one click creates exactly one request with `{ marketId, chainId, maxAgeSeconds: 300 }`;
3. the action is disabled while pending, so duplicate clicks do not create another request;
4. a complete valid response replaces loading;
5. an old complete thesis remains visible until a replacement response fully validates;
6. unmount and identity change abort the request;
7. a late response for an earlier identity is ignored;
8. one logical retry reuses its idempotency key after an ambiguous network failure;
9. a fresh explicit request after success creates a new key;
10. time reaching `expiresAt` moves the artifact to `expired` while preserving evidence;
11. `429` never triggers an automatic retry;
12. `409` calls the provided market-refresh action only after the user clicks `Refresh market`.

The no-auto-request assertion starts as:

```ts
const createThesisSpy = vi.spyOn(api, "createThesis");
render(<ThesisPanelController chainId={50312} eligible marketId="fixture-btc-18utc" marketObservationKey="2026-08-28T15:46:00.000Z" onRefreshMarket={onRefreshMarket} />);
expect(createThesisSpy).not.toHaveBeenCalled();
```

- [ ] **Step 2: Run the controller test and verify RED**

```bash
pnpm --filter @oraclelane/web exec vitest run src/features/case-file/thesis-panel-controller.test.tsx
```

Expected: FAIL because the controller does not exist.

- [ ] **Step 3: Implement an explicit-action controller**

Use this boundary:

```ts
type ThesisPanelControllerProps = Readonly<{
  marketId: string;
  chainId: number;
  marketObservationKey: string;
  eligible: boolean;
  eligibilityReason?: string;
  onRefreshMarket: () => Promise<boolean>;
}>;
```

Store state immutably. Use `AbortController`, a monotonic request sequence ref, and an identity string composed from `chainId`, `marketId`, and `marketObservationKey`. Cleanup must abort the active request.

Keep the most recent complete validated `ThesisResponse` separately from the in-flight/error status. Pass it as `previousResponse` during loading or failure so a new request cannot erase audit evidence before a replacement validates. Clear it only when the market identity changes. Reclassify that preserved response as expired when its own `expiresAt` is reached.

Generate keys from browser cryptography:

```ts
function createIdempotencyKey() {
  return `oraclelane:${crypto.randomUUID()}`;
}
```

Do not use timestamps or `Math.random()` for idempotency keys. Preserve the key for a retry of the same unresolved logical request; clear it after a complete valid response or market identity change.

- [ ] **Step 4: Add a bounded expiry clock**

Update `now` while a successful thesis is unexpired. Stop the timer for all non-success states and after expiry. Use at most one interval and clean it up on state or route change.

The UI may show a changing remaining time, but the canonical `expiresAt` remains unchanged.

- [ ] **Step 5: Implement explicit recovery policies**

- `market_conflict`: call `onRefreshMarket`; reset to `not_requested` only after refresh succeeds.
- `rate_limited`: display the bounded wait; enable user retry only when the local wait reaches zero; never schedule the network request automatically.
- `rejected`: do not retry blindly; offer `Build another thesis` only as a new explicit operation with a new key.
- `unavailable`: allow one user-triggered retry with the same logical key; further failure remains explicit and does not loop.

- [ ] **Step 6: Run controller and state tests**

```bash
pnpm --filter @oraclelane/web exec vitest run \
  src/features/case-file/thesis-panel-controller.test.tsx \
  src/features/case-file/thesis-panel.test.tsx \
  src/features/case-file/thesis-panel-model.test.ts
pnpm typecheck
git diff --check
```

Expected: lifecycle tests pass with no open timers, unhandled promises, duplicate requests, or stale response replacement.

- [ ] **Step 7: Commit Task 4**

```bash
git add apps/web/src/features/case-file/thesis-panel-controller.tsx apps/web/src/features/case-file/thesis-panel-controller.test.tsx apps/web/src/features/case-file/thesis-panel-model.ts
git commit -m "feat: control thesis request lifecycle"
```

---

## Task 5: Integrate Slice 2 into the Market Case File

**Files:**

- Modify: `apps/web/src/features/case-file/case-file-sections.tsx`
- Modify: `apps/web/src/features/case-file/case-file-surface.tsx`
- Modify: `apps/web/src/features/case-file/case-file-surface.test.tsx`
- Modify: `apps/web/src/features/case-file/case-file.css`

- [ ] **Step 1: Replace the deferred-button assertions with failing Slice 2 assertions**

Update the Case File surface tests to prove:

- the existing open/fresh market shows an enabled `Build a thesis` action;
- stale, unknown-freshness, paused, locked, resolved, and void markets keep the action disabled with an exact reason;
- Case File question, outcomes, timing, terms, and provenance remain visible while thesis loading or failed;
- market refresh success resets a thesis tied to the earlier observation;
- market refresh failure preserves both the last valid market and the last complete thesis;
- no wallet or trade action appears.

Keep the existing Case File fact assertions; do not weaken them to make Slice 2 pass.

- [ ] **Step 2: Run the Case File test and verify RED**

```bash
pnpm --filter @oraclelane/web exec vitest run src/features/case-file/case-file-surface.test.tsx
```

Expected: FAIL because `Read thesis` is still disabled and no controller is mounted.

- [ ] **Step 3: Remove the old deferred section**

Delete `CaseFileDeferredNextStep` from `case-file-sections.tsx` and its imports/usages. Do not leave duplicate `Read thesis` and `Build a thesis` controls.

- [ ] **Step 4: Mount the controller after Case File evidence**

In `case-file-surface.tsx`, calculate eligibility from the current typed market:

```ts
const thesisEligible = market.status === "OPEN" && market.freshness === "FRESH";
const thesisEligibilityReason = market.status !== "OPEN"
  ? "A thesis is available only while the market is open."
  : market.freshness !== "FRESH"
    ? "Refresh to a fresh market observation before building a thesis."
    : undefined;
```

Mount:

```tsx
<ThesisPanelController
  chainId={chainId}
  eligible={thesisEligible}
  eligibilityReason={thesisEligibilityReason}
  marketId={marketId}
  marketObservationKey={market.observedAt}
  onRefreshMarket={refreshMarket}
/>
```

Change `refreshMarket` to return `Promise<boolean>` without leaking upstream errors. It returns `true` only after a complete typed `MarketResponse` replaces the prior response.

- [ ] **Step 5: Add responsive panel styling using existing tokens**

Append feature-local selectors to `case-file.css`. Use only established tokens such as `--background`, `--foreground`, `--card`, `--muted`, `--muted-foreground`, `--border`, `--destructive`, `--space-*`, and `--radius-*`.

Required layout behavior:

- desktop: summary first, drivers and risks in two balanced columns, evidence below;
- mobile below 768 px: one column, full-width actions, wrapping evidence URLs;
- visible focus states come from shadcn/ui primitives;
- no horizontal overflow at 320 px;
- reduced motion follows the existing global media query;
- advisory visuals use neutral/accent tokens and never imitate a guaranteed signal.

- [ ] **Step 6: Run the complete Case File suite**

```bash
pnpm --filter @oraclelane/web exec vitest run \
  src/features/case-file/case-file-model.test.ts \
  src/features/case-file/case-file-sections.test.tsx \
  src/features/case-file/case-file-surface.test.tsx \
  src/features/case-file/thesis-panel-model.test.ts \
  src/features/case-file/thesis-panel.test.tsx \
  src/features/case-file/thesis-panel-controller.test.tsx \
  'src/app/markets/[chainId]/[marketId]/page.test.tsx'
pnpm lint
pnpm typecheck
git diff --check
```

Expected: all Case File and Thesis Panel tests pass with no regression to route parsing, market refresh, or typed unavailable fields.

- [ ] **Step 7: Commit Task 5**

```bash
git add apps/web/src/features/case-file
git commit -m "feat: integrate thesis panel into case file"
```

---

## Task 6: Add a browser-level critical-flow gate

**Files:**

- Modify: `apps/web/package.json`
- Modify: `pnpm-lock.yaml`
- Create: `apps/web/playwright.config.ts`
- Create: `apps/web/e2e/thesis-panel.spec.ts`
- Modify: `.github/workflows/ci.yml`

- [ ] **Step 1: Add Playwright and scripts**

```bash
cd /Users/mac/developer/oraclelane
pnpm --filter @oraclelane/web add -D @playwright/test
```

Add:

```json
{
  "scripts": {
    "test:e2e": "playwright test"
  }
}
```

- [ ] **Step 2: Configure a fixture-mode Next.js web server**

Create `apps/web/playwright.config.ts` with Chromium, base URL `http://127.0.0.1:3000`, screenshots/traces on first retry, and:

```ts
webServer: {
  command: "pnpm dev",
  url: "http://127.0.0.1:3000/radar",
  reuseExistingServer: !process.env.CI,
  env: {
    ...process.env,
    ORACLEANE_API_MODE: "fixture",
    NEXT_PUBLIC_ORACLEANE_API_MODE: "fixture",
  },
},
```

- [ ] **Step 3: Write the failing Radar-to-thesis E2E test**

Create `apps/web/e2e/thesis-panel.spec.ts`:

```ts
import { expect, test } from "@playwright/test";

test("user moves from Radar to a complete advisory thesis", async ({ page }) => {
  const consoleErrors: string[] = [];
  page.on("console", (message) => {
    if (message.type() === "error") consoleErrors.push(message.text());
  });
  await page.goto("/radar");
  await page.getByRole("link", { name: "Read case file" }).click();

  await expect(page.getByRole("heading", { level: 1, name: /Will BTC be above/ })).toBeVisible();
  await page.getByRole("button", { name: "Build a thesis" }).click();

  await expect(page.getByText("Oraclelane thesis · advisory, not a guarantee")).toBeVisible();
  await expect(page.getByText("UP", { exact: true })).toBeVisible();
  await expect(page.getByText("Medium confidence")).toBeVisible();
  await expect(page.getByRole("list", { name: "Thesis drivers" })).toBeVisible();
  await expect(page.getByRole("list", { name: "Thesis risks" })).toBeVisible();
  await expect(page.getByRole("link", { name: "DreamDEX Event Contract market snapshot" })).toHaveAttribute("href", /^https:\/\//);
  await expect(page.getByRole("button", { name: /connect wallet|trade|redeem|sign/i })).toHaveCount(0);
  expect(consoleErrors).toEqual([]);
});
```

- [ ] **Step 4: Run and make the browser flow GREEN**

```bash
pnpm --filter @oraclelane/web exec playwright install chromium
pnpm --filter @oraclelane/web test:e2e
```

Expected: Chromium completes the real Radar → Case File → explicit Thesis Panel flow with no console error and no transaction action.

- [ ] **Step 5: Add E2E to CI**

After unit tests in `.github/workflows/ci.yml`, add:

```yaml
      - run: pnpm --filter @oraclelane/web exec playwright install --with-deps chromium
        working-directory: .
      - run: pnpm --filter @oraclelane/web test:e2e
        working-directory: .
```

- [ ] **Step 6: Run unit coverage and E2E together**

```bash
pnpm --filter @oraclelane/web test:coverage
pnpm --filter @oraclelane/web test:e2e
git diff --check
```

Expected: all coverage thresholds are at least 80%, the critical E2E passes, and no browser artifact is accidentally tracked.

- [ ] **Step 7: Commit Task 6**

```bash
git add apps/web/package.json pnpm-lock.yaml apps/web/playwright.config.ts apps/web/e2e/thesis-panel.spec.ts .github/workflows/ci.yml
git commit -m "test: cover thesis critical flow"
```

---

## Task 7: Update delivery documentation and run the final gate

**Files:**

- Modify: `README.md`
- Modify: `apps/web/README.md`
- Modify: `architecture/docs/24-frontend-slicing-plan.md`

- [ ] **Step 1: Update app documentation**

Document Slice 2 as:

- explicit advisory request inside the Case File;
- generated OpenAPI request/response boundary;
- complete `UP`, `DOWN`, and `NO_TRADE` states;
- evidence, generated time, expiry, model version, and prompt version;
- safe `409/429/502/503` recovery;
- still no wallet, transaction, trade preview, or redeem path.

Add the focused local commands:

```bash
pnpm generate:api
pnpm test
pnpm --filter @oraclelane/web test:e2e
```

- [ ] **Step 2: Mark only implemented Slice 2 items complete**

Update `architecture/docs/24-frontend-slicing-plan.md` only after the application work and tests pass. Do not mark backend model orchestration or production DreamDEX integration complete if the frontend is still using fixtures.

- [ ] **Step 3: Run the final verification gate**

```bash
cd /Users/mac/developer/oraclelane
pnpm generate:api
git diff --exit-code -- apps/web/src/lib/api/generated.ts
pnpm lint
pnpm typecheck
pnpm test
pnpm --filter @oraclelane/web test:coverage
pnpm build
pnpm --filter @oraclelane/web test:e2e
pnpm audit --audit-level=high
git diff --check
git status --short
```

Expected:

- generated API types have no drift;
- lint and typecheck pass;
- all unit/integration tests pass;
- coverage remains at least 80% for statements, branches, functions, and lines;
- production build passes;
- Chromium E2E passes;
- no high or critical audit advisory is reported;
- only intentional documentation/application changes remain.

- [ ] **Step 4: Perform the security and scope review**

Confirm from the final diff:

- no secret, provider key, prompt, raw model output, or stack trace is committed;
- no `dangerouslySetInnerHTML` was added to Thesis Panel code;
- no wallet or transaction dependency was added;
- every network request is explicit and abortable;
- rate limit and idempotency semantics remain visible and tested;
- unknown or partial responses cannot reach the artifact renderer;
- all external evidence links are HTTPS with safe new-tab attributes.

- [ ] **Step 5: Commit documentation in the architecture repository**

```bash
cd /Users/mac/developer/oraclelane/architecture
git add docs/24-frontend-slicing-plan.md docs/superpowers/plans/2026-08-30-slice-2-thesis-panel.md
git commit -m "docs: add Slice 2 implementation plan"
```

- [ ] **Step 6: Commit application documentation and submodule pointer**

```bash
cd /Users/mac/developer/oraclelane
git add README.md apps/web/README.md architecture
git commit -m "docs: record Slice 2 frontend delivery"
```

Do not push either repository until the user explicitly authorizes publication of the completed implementation.

---

## Final acceptance checklist

- [ ] A user reaches the Case File from Radar and explicitly requests a thesis.
- [ ] No thesis request occurs on page load.
- [ ] The request contains only market identity and `maxAgeSeconds`, plus the required idempotency header.
- [ ] Successful payloads validate strictly before rendering and remain bound to the current market identity.
- [ ] `UP`, `DOWN`, and `NO_TRADE` render as complete advisory artifacts.
- [ ] Evidence, generated time, expiry, model version, prompt version, market version, and cache status are visible.
- [ ] Expired evidence remains visible, but the artifact is marked unusable for a later preview.
- [ ] `409`, `429`, `502`, `503`, and safe network failures produce distinct recovery behavior.
- [ ] Duplicate clicks, late responses, route changes, and unmounts cannot corrupt state.
- [ ] Case File facts remain usable during loading and failure.
- [ ] The panel uses official shadcn/ui primitives and the accepted Oraclelane tokens.
- [ ] No wallet, trade, sign, redeem, market creation, pooled custody, or autonomous execution path exists.
- [ ] Unit, integration, coverage, build, security audit, and Chromium E2E gates pass.

## Plan self-review record

- Spec coverage: every state and invariant in `docs/27-thesis-panel-architecture.md` is mapped to a task and test.
- Contract consistency: request, response, error status, idempotency, expiry, and identity fields match OpenAPI v1.1.0 generated types.
- Placeholder scan: the plan contains no `TBD`, `TODO`, ellipsis placeholder, invented API field, or unresolved file path.
- Scope check: frontend Slice 2 only; production backend orchestration, wallet, safety gate, trade preview, and execution remain deferred.
