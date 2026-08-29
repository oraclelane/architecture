# Figma-aligned Slice 0 Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the existing `/radar` Slice 0 surface to match the approved desktop composition while using the exact Oraclelane Figma foundation tokens, preserving the typed API boundary, and providing real responsive filter and refresh interactions.

> **Delivery naming:** This plan ships as **Slice 0 foundation + radar preview**. The radar surface is included to prove the shell, typed read boundary, filter state, and refresh behavior end to end; it does not represent the full Market Radar product slice. Market Case File is the next product feature. Wallet, trade, and redeem remain deferred to their own approved slices.

**Architecture:** Keep `/radar` as a server entry that fetches the initial typed `MarketListResponse`, then hand interaction ownership to a client-side `RadarSurface`. Keep global navigation in `AppShell`; place filter state, desktop rail, mobile dialog, semantic market list, refresh state, and filtered-empty state inside the radar feature. Separate exact design tokens from layout CSS so foundation fidelity can be tested directly.

**Tech Stack:** Next.js 15 App Router, React 19, TypeScript 5.8, CSS custom properties, Zod-validated OpenAPI types, Vitest 3, Testing Library, MSW.

---

## Source of truth and repository boundary

- Approved design spec: `architecture/docs/superpowers/specs/2026-08-29-figma-aligned-slice-0-redesign.md`
- Figma foundation: [Oraclelane node 4:2](https://www.figma.com/design/JgdpTiCVtDE6KsYB0BjENy/Oraclelane?node-id=4-2&p=f)
- Approved composition reference: `/Users/mac/.codex/generated_images/01a047d7-c6d5-71e1-99fa-81eb1bd1a3db/exec-9a673f15-cb84-4cbe-8f47-fcb271168d5b.png`
- Application workspace: `/Users/mac/developer/oraclelane`
- Documentation repository: `/Users/mac/developer/oraclelane/architecture`

The application workspace is not currently a Git worktree. Each task therefore includes a proposed application commit. Run that commit only if `/Users/mac/developer/oraclelane` has been connected to the intended Git repository before execution. Otherwise record the proposed commit in the execution report and continue; never commit application files into the nested `architecture` or `blueprint` repositories.

## Planned file map

**Modify**

- `apps/web/src/app/layout.tsx`
- `apps/web/src/app/radar/page.tsx`
- `apps/web/src/app/loading.tsx`
- `apps/web/src/components/app-shell.tsx`
- `apps/web/src/components/app-shell.test.tsx`
- `apps/web/src/components/providers/theme-provider.tsx`
- `apps/web/src/components/providers/theme-provider.test.tsx`
- `apps/web/src/lib/utils.ts`
- `apps/web/src/lib/utils.test.ts`
- `apps/web/src/styles/globals.css`
- `apps/web/vitest.config.ts`

**Create**

- `apps/web/src/components/providers/theme-bootstrap.tsx`
- `apps/web/src/components/providers/theme-bootstrap.test.ts`
- `apps/web/src/styles/foundation.css`
- `apps/web/src/styles/foundation.test.ts`
- `apps/web/src/features/radar/radar-model.ts`
- `apps/web/src/features/radar/radar-model.test.ts`
- `apps/web/src/features/radar/filter-controls.tsx`
- `apps/web/src/features/radar/filter-controls.test.tsx`
- `apps/web/src/features/radar/filter-sheet.tsx`
- `apps/web/src/features/radar/filter-sheet.test.tsx`
- `apps/web/src/features/radar/market-list.tsx`
- `apps/web/src/features/radar/market-list.test.tsx`
- `apps/web/src/features/radar/radar-surface.tsx`
- `apps/web/src/features/radar/radar-surface.test.tsx`
- `apps/web/src/features/radar/radar.css`

**Delete after replacement tests pass**

- `apps/web/src/features/foundation/market-fixture.tsx`
- `apps/web/src/features/foundation/market-fixture.test.tsx`

## Task 1: Lock the exact Figma foundation and pre-hydration theme

**Files:**

- Create: `apps/web/src/styles/foundation.css`
- Create: `apps/web/src/styles/foundation.test.ts`
- Create: `apps/web/src/components/providers/theme-bootstrap.tsx`
- Create: `apps/web/src/components/providers/theme-bootstrap.test.ts`
- Modify: `apps/web/src/components/providers/theme-provider.tsx`
- Modify: `apps/web/src/components/providers/theme-provider.test.tsx`
- Modify: `apps/web/src/app/layout.tsx`
- Modify: `apps/web/src/styles/globals.css`
- Modify: `apps/web/vitest.config.ts`

- [ ] **Step 1: Write a failing exact-token test**

Create `apps/web/src/styles/foundation.test.ts`:

```ts
// @vitest-environment node

import { readFileSync } from "node:fs";
import { resolve } from "node:path";
import { describe, expect, it } from "vitest";

const css = readFileSync(resolve(process.cwd(), "src/styles/foundation.css"), "utf8");

describe("Figma foundation tokens", () => {
  it("contains the exact light palette", () => {
    expect(css).toContain("--background: #ffffff;");
    expect(css).toContain("--foreground: #09090b;");
    expect(css).toContain("--card: #ffffff;");
    expect(css).toContain("--primary: #18181b;");
    expect(css).toContain("--secondary: #f4f4f5;");
    expect(css).toContain("--destructive: #e7000b;");
    expect(css).toContain("--border: #e4e4e7;");
    expect(css).toContain("--ring: #9f9fa9;");
    expect(css).toContain("--sidebar: #fafafa;");
    expect(css).toContain("--sidebar-primary: #18181b;");
  });

  it("contains the exact dark palette and foundation scales", () => {
    expect(css).toContain("--background: #09090b;");
    expect(css).toContain("--foreground: #fafafa;");
    expect(css).toContain("--card: #18181b;");
    expect(css).toContain("--primary: #e4e4e7;");
    expect(css).toContain("--destructive: #ff6467;");
    expect(css).toContain("--border: rgba(255, 255, 255, 0.1);");
    expect(css).toContain("--sidebar-primary: #1447e6;");
    expect(css).toContain("--space-10: 64px;");
    expect(css).toContain("--radius-sm: 3.2px;");
    expect(css).toContain("--radius-xl: 11.2px;");
  });
});
```

- [ ] **Step 2: Run the token test and verify RED**

Run:

```bash
cd /Users/mac/developer/oraclelane
pnpm --filter @oraclelane/web test -- src/styles/foundation.test.ts
```

Expected: FAIL because `src/styles/foundation.css` does not exist.

- [ ] **Step 3: Add the exact foundation file**

Create `apps/web/src/styles/foundation.css`:

```css
:root {
  color-scheme: light;
  --background: #ffffff;
  --foreground: #09090b;
  --card: #ffffff;
  --card-foreground: #09090b;
  --popover: #ffffff;
  --popover-foreground: #09090b;
  --primary: #18181b;
  --primary-foreground: #fafafa;
  --secondary: #f4f4f5;
  --secondary-foreground: #18181b;
  --muted: #f4f4f5;
  --muted-foreground: #71717b;
  --accent: #f4f4f5;
  --accent-foreground: #18181b;
  --destructive: #e7000b;
  --border: #e4e4e7;
  --input: #e4e4e7;
  --ring: #9f9fa9;
  --chart-1: #d4d4d8;
  --chart-2: #71717b;
  --chart-3: #52525c;
  --chart-4: #3f3f46;
  --chart-5: #27272a;
  --sidebar: #fafafa;
  --sidebar-primary: #18181b;
  --success: #15803d;
  --warning: #a16207;
  --advisory: #6d28d9;
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-7: 32px;
  --space-8: 40px;
  --space-9: 48px;
  --space-10: 64px;
  --radius-sm: 3.2px;
  --radius-md: 5.2px;
  --radius-lg: 7.2px;
  --radius-xl: 11.2px;
  --font-sans: "Inter", ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-mono: "JetBrains Mono", "SFMono-Regular", Consolas, monospace;
}

.dark {
  color-scheme: dark;
  --background: #09090b;
  --foreground: #fafafa;
  --card: #18181b;
  --card-foreground: #fafafa;
  --popover: #18181b;
  --popover-foreground: #fafafa;
  --primary: #e4e4e7;
  --primary-foreground: #18181b;
  --secondary: #27272a;
  --secondary-foreground: #fafafa;
  --muted: #27272a;
  --muted-foreground: #a1a1aa;
  --accent: #27272a;
  --accent-foreground: #fafafa;
  --destructive: #ff6467;
  --border: rgba(255, 255, 255, 0.1);
  --input: rgba(255, 255, 255, 0.15);
  --ring: #71717b;
  --chart-1: #d4d4d8;
  --chart-2: #71717b;
  --chart-3: #52525c;
  --chart-4: #3f3f46;
  --chart-5: #27272a;
  --sidebar: #18181b;
  --sidebar-primary: #1447e6;
  --success: #4ade80;
  --warning: #facc15;
  --advisory: #c4b5fd;
}
```

- [ ] **Step 4: Write the pre-hydration bootstrap test**

Create `apps/web/src/components/providers/theme-bootstrap.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { THEME_BOOTSTRAP_SCRIPT } from "./theme-bootstrap";

describe("theme bootstrap", () => {
  it("reads the persisted Oraclelane theme before hydration", () => {
    expect(THEME_BOOTSTRAP_SCRIPT).toContain("oraclelane-theme");
    expect(THEME_BOOTSTRAP_SCRIPT).toContain("classList.toggle");
    expect(THEME_BOOTSTRAP_SCRIPT).toContain("dataset.theme");
  });
});
```

Run:

```bash
pnpm --filter @oraclelane/web test -- src/components/providers/theme-bootstrap.test.ts
```

Expected: FAIL because the module does not exist.

- [ ] **Step 5: Add the bootstrap and align provider initialization**

Create `apps/web/src/components/providers/theme-bootstrap.tsx`:

```tsx
export const THEME_BOOTSTRAP_SCRIPT = `(() => {
  try {
    const storedTheme = window.localStorage.getItem("oraclelane-theme");
    const theme = storedTheme === "light" || storedTheme === "dark" ? storedTheme : "dark";
    document.documentElement.classList.toggle("dark", theme === "dark");
    document.documentElement.dataset.theme = theme;
  } catch {
    document.documentElement.classList.add("dark");
    document.documentElement.dataset.theme = "dark";
  }
})();`;

export function ThemeBootstrap() {
  return <script dangerouslySetInnerHTML={{ __html: THEME_BOOTSTRAP_SCRIPT }} />;
}
```

In `ThemeProvider`, replace the initial state with a lazy, browser-aware initializer and keep immutable functional updates:

```tsx
const [theme, setThemeState] = useState<Theme>(() => {
  if (typeof window === "undefined") return defaultTheme;
  try {
    const storedTheme = window.localStorage.getItem(storageKey);
    return storedTheme === "light" || storedTheme === "dark" ? storedTheme : defaultTheme;
  } catch {
    return defaultTheme;
  }
});
```

Delete the effect that only reads storage. Keep the effect that applies the class, dataset, and persisted value.

In `layout.tsx`, import and render `<ThemeBootstrap />` inside `<head>` before `<body>`. Keep `suppressHydrationWarning` on `<html>` and retain the default `dark` class.

At the first line of `globals.css`, add:

```css
@import "./foundation.css";
```

Remove the old `:root` and `.dark` token blocks from `globals.css`. Replace all direct font stacks there with `var(--font-sans)` or `var(--font-mono)`, remove the radial gradient and all decorative card shadows, and change every radius to one of the four foundation radius variables.

Add `src/styles/**/*.ts` to the Vitest coverage exclude list because token assertion files are tests, not runtime source:

```ts
exclude: [
  "src/lib/api/generated.ts",
  "src/lib/api/types.ts",
  "src/mocks/browser.ts",
  "src/mocks/server.ts",
  "src/styles/**/*.ts",
],
```

- [ ] **Step 6: Run focused tests and verify GREEN**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/styles/foundation.test.ts src/components/providers/theme-bootstrap.test.ts src/components/providers/theme-provider.test.tsx
```

Expected: all focused tests pass; light removes `.dark`, dark restores it, and persisted state is used on first client render.

- [ ] **Step 7: Proposed application commit**

If the workspace is a Git worktree:

```bash
git add apps/web/src/styles/foundation.css apps/web/src/styles/foundation.test.ts apps/web/src/styles/globals.css apps/web/src/components/providers/theme-bootstrap.tsx apps/web/src/components/providers/theme-bootstrap.test.ts apps/web/src/components/providers/theme-provider.tsx apps/web/src/components/providers/theme-provider.test.tsx apps/web/src/app/layout.tsx apps/web/vitest.config.ts
git commit -m "feat: align Slice 0 theme with Figma foundation"
```

## Task 2: Make the header the only global navigation surface

**Files:**

- Modify: `apps/web/src/components/app-shell.tsx`
- Modify: `apps/web/src/components/app-shell.test.tsx`
- Modify: `apps/web/src/styles/globals.css`

- [ ] **Step 1: Extend the shell test to fail on duplicate navigation**

Add these assertions to the primary navigation test:

```tsx
expect(screen.getAllByRole("navigation")).toHaveLength(1);
expect(screen.getByRole("link", { name: "Market radar" })).toHaveAttribute("aria-current", "page");
expect(screen.queryByRole("complementary", { name: "Workspace navigation" })).not.toBeInTheDocument();
expect(screen.queryByText("Decision terminal")).not.toBeInTheDocument();
```

- [ ] **Step 2: Run the test and verify RED**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/components/app-shell.test.tsx
```

Expected: FAIL because the old workspace rail is present and active links do not expose `aria-current`.

- [ ] **Step 3: Refactor `AppShell` into header, single main, and footer**

Keep `navigation`, `ThemeToggle`, the wallet notice, and route-aware classes. For each active link add:

```tsx
aria-current={active ? "page" : undefined}
```

Replace the body grid with:

```tsx
<main className="page-container">{children}</main>
```

Delete the entire `workspace-rail` aside. Keep the footer outside `<main>`.

Update `globals.css` so:

```css
.page-container {
  width: min(1440px, calc(100% - 48px));
  min-width: 0;
  flex: 1;
  margin: 0 auto;
  padding: var(--space-10) 0 80px;
}

.nav-link[aria-current="page"] {
  color: var(--foreground);
  background: var(--accent);
}
```

Delete `.shell-body`, `.workspace-rail`, `.rail-label`, `.rail-title`, `.rail-copy`, `.rail-status`, and `.status-dot` rules and their breakpoint overrides. At mobile width set `.page-container` to `width: min(100% - 32px, 620px)`.

- [ ] **Step 4: Run the shell tests and verify GREEN**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/components/app-shell.test.tsx
```

Expected: all tests pass, exactly one navigation landmark exists, and no workspace rail is rendered.

- [ ] **Step 5: Proposed application commit**

```bash
git add apps/web/src/components/app-shell.tsx apps/web/src/components/app-shell.test.tsx apps/web/src/styles/globals.css
git commit -m "refactor: keep Oraclelane navigation in the header"
```

## Task 3: Build the immutable radar filter and sort model

**Files:**

- Create: `apps/web/src/features/radar/radar-model.ts`
- Create: `apps/web/src/features/radar/radar-model.test.ts`
- Modify: `apps/web/src/lib/utils.ts`
- Modify: `apps/web/src/lib/utils.test.ts`

- [ ] **Step 1: Write the pure-model tests**

Create `apps/web/src/features/radar/radar-model.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import type { Market } from "@/lib/api/types";
import {
  DEFAULT_RADAR_FILTERS,
  inferMarketAsset,
  selectRadarMarkets,
  type RadarFilters,
} from "./radar-model";

const market = (overrides: Partial<Market>): Market => ({
  marketId: "market-btc",
  chainId: 50312,
  question: "Will BTC close above $68,000?",
  status: "OPEN",
  outcomes: [],
  closeAt: "2026-08-29T18:00:00.000Z",
  observedAt: "2026-08-29T15:00:00.000Z",
  freshness: "FRESH",
  ...overrides,
});

describe("radar model", () => {
  it("classifies only word-boundary BTC and ETH symbols", () => {
    expect(inferMarketAsset("Will BTC rise?")).toBe("BTC");
    expect(inferMarketAsset("Will eth rise?")).toBe("ETH");
    expect(inferMarketAsset("Will XBTC rise?")).toBe("OTHER");
    expect(inferMarketAsset("Will SOL rise?")).toBe("OTHER");
  });

  it("filters without mutating the API array and sorts by close time", () => {
    const input = [
      market({ marketId: "eth", question: "Will ETH rise?", closeAt: "2026-08-29T20:00:00.000Z" }),
      market({ marketId: "btc", closeAt: "2026-08-29T17:00:00.000Z" }),
    ];
    const snapshot = [...input];
    const filters: RadarFilters = { ...DEFAULT_RADAR_FILTERS, asset: "BTC" };

    expect(selectRadarMarkets(input, filters).map(({ marketId }) => marketId)).toEqual(["btc"]);
    expect(input).toEqual(snapshot);
  });

  it("supports real status and alternate observed-time sorting", () => {
    const input = [
      market({ marketId: "open", observedAt: "2026-08-29T14:00:00.000Z" }),
      market({ marketId: "paused", status: "PAUSED", observedAt: "2026-08-29T16:00:00.000Z" }),
    ];
    const filters: RadarFilters = {
      asset: "ALL",
      status: "PAUSED",
      sort: "OBSERVED_TIME",
    };

    expect(selectRadarMarkets(input, filters).map(({ marketId }) => marketId)).toEqual(["paused"]);
  });
});
```

- [ ] **Step 2: Run the model test and verify RED**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/radar-model.test.ts
```

Expected: FAIL because the model module does not exist.

- [ ] **Step 3: Implement the minimal immutable model**

Create `apps/web/src/features/radar/radar-model.ts`:

```ts
import type { Market, MarketStatus } from "@/lib/api/types";

export type AssetFilter = "ALL" | "BTC" | "ETH";
export type SortFilter = "CLOSE_TIME" | "OBSERVED_TIME";

export type RadarFilters = Readonly<{
  asset: AssetFilter;
  status: MarketStatus;
  sort: SortFilter;
}>;

export const DEFAULT_RADAR_FILTERS: RadarFilters = {
  asset: "ALL",
  status: "OPEN",
  sort: "CLOSE_TIME",
};

export function inferMarketAsset(question: string): Exclude<AssetFilter, "ALL"> | "OTHER" {
  const normalizedQuestion = question.toUpperCase();
  if (/\bBTC\b/.test(normalizedQuestion)) return "BTC";
  if (/\bETH\b/.test(normalizedQuestion)) return "ETH";
  return "OTHER";
}

export function selectRadarMarkets(markets: readonly Market[], filters: RadarFilters): Market[] {
  const filtered = markets.filter((market) => {
    const assetMatches = filters.asset === "ALL" || inferMarketAsset(market.question) === filters.asset;
    return assetMatches && market.status === filters.status;
  });
  const timestampKey = filters.sort === "CLOSE_TIME" ? "closeAt" : "observedAt";
  return [...filtered].sort(
    (left, right) => new Date(left[timestampKey]).getTime() - new Date(right[timestampKey]).getTime(),
  );
}
```

- [ ] **Step 4: Add deterministic UTC formatters test-first**

Add to `apps/web/src/lib/utils.test.ts`:

```ts
import { formatUtcDateTime, latestObservedAt } from "./utils";

it("formats operational timestamps in UTC", () => {
  expect(formatUtcDateTime("2026-08-29T15:46:00.000Z")).toBe("29 Aug 2026, 15:46 UTC");
});

it("finds the latest observed timestamp", () => {
  expect(latestObservedAt([
    { observedAt: "2026-08-29T14:00:00.000Z" },
    { observedAt: "2026-08-29T16:00:00.000Z" },
  ])).toBe("2026-08-29T16:00:00.000Z");
});
```

Implement in `utils.ts`:

```ts
export function formatUtcDateTime(value: string) {
  return `${new Intl.DateTimeFormat("en-GB", {
    day: "2-digit",
    month: "short",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    hour12: false,
    timeZone: "UTC",
  }).format(new Date(value))} UTC`;
}

export function latestObservedAt(values: readonly { observedAt: string }[]) {
  return values.reduce<string | null>((latest, value) => {
    if (latest === null) return value.observedAt;
    return new Date(value.observedAt).getTime() > new Date(latest).getTime() ? value.observedAt : latest;
  }, null);
}
```

- [ ] **Step 5: Run model and utility tests**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/radar-model.test.ts src/lib/utils.test.ts
```

Expected: all focused tests pass and no input array is mutated.

- [ ] **Step 6: Proposed application commit**

```bash
git add apps/web/src/features/radar/radar-model.ts apps/web/src/features/radar/radar-model.test.ts apps/web/src/lib/utils.ts apps/web/src/lib/utils.test.ts
git commit -m "feat: add immutable radar filtering model"
```

## Task 4: Build shared desktop and mobile filter controls

**Files:**

- Create: `apps/web/src/features/radar/filter-controls.tsx`
- Create: `apps/web/src/features/radar/filter-controls.test.tsx`
- Create: `apps/web/src/features/radar/filter-sheet.tsx`
- Create: `apps/web/src/features/radar/filter-sheet.test.tsx`
- Create: `apps/web/src/features/radar/radar.css`

- [ ] **Step 1: Write a failing shared-controls test**

Create `apps/web/src/features/radar/filter-controls.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it, vi } from "vitest";
import { DEFAULT_RADAR_FILTERS } from "./radar-model";
import { FilterControls } from "./filter-controls";

describe("FilterControls", () => {
  it("updates asset, status, sort, and reset through one controlled contract", async () => {
    const user = userEvent.setup();
    const onChange = vi.fn();
    const onReset = vi.fn();
    render(<FilterControls filters={DEFAULT_RADAR_FILTERS} onChange={onChange} onReset={onReset} />);

    await user.click(screen.getByRole("radio", { name: "BTC" }));
    expect(onChange).toHaveBeenCalledWith({ ...DEFAULT_RADAR_FILTERS, asset: "BTC" });

    await user.selectOptions(screen.getByLabelText("Status"), "PAUSED");
    expect(onChange).toHaveBeenCalledWith({ ...DEFAULT_RADAR_FILTERS, status: "PAUSED" });

    await user.selectOptions(screen.getByLabelText("Sort"), "OBSERVED_TIME");
    expect(onChange).toHaveBeenCalledWith({ ...DEFAULT_RADAR_FILTERS, sort: "OBSERVED_TIME" });

    await user.click(screen.getByRole("button", { name: "Reset filters" }));
    expect(onReset).toHaveBeenCalledOnce();
  });
});
```

- [ ] **Step 2: Run the controls test and verify RED**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/filter-controls.test.tsx
```

Expected: FAIL because `FilterControls` does not exist.

- [ ] **Step 3: Implement the controlled filter form**

Create `apps/web/src/features/radar/filter-controls.tsx`:

```tsx
import type { MarketStatus } from "@/lib/api/types";
import type { AssetFilter, RadarFilters, SortFilter } from "./radar-model";

const assets: readonly AssetFilter[] = ["ALL", "BTC", "ETH"];
const statuses: readonly MarketStatus[] = ["OPEN", "PAUSED", "LOCKED", "RESOLVED", "VOID"];

type FilterControlsProps = Readonly<{
  filters: RadarFilters;
  onChange: (filters: RadarFilters) => void;
  onReset: () => void;
}>;

export function FilterControls({ filters, onChange, onReset }: FilterControlsProps) {
  return (
    <form aria-label="Market filters" className="filter-form" onSubmit={(event) => event.preventDefault()}>
      <fieldset className="filter-group">
        <legend>Asset</legend>
        <div className="asset-options">
          {assets.map((asset) => (
            <label className="asset-option" key={asset}>
              <input
                checked={filters.asset === asset}
                name="asset"
                onChange={() => onChange({ ...filters, asset })}
                type="radio"
              />
              <span>{asset === "ALL" ? "All" : asset}</span>
            </label>
          ))}
        </div>
      </fieldset>

      <label className="filter-field">
        <span>Status</span>
        <select
          onChange={(event) => onChange({ ...filters, status: event.target.value as MarketStatus })}
          value={filters.status}
        >
          {statuses.map((status) => <option key={status} value={status}>{status[0]}{status.slice(1).toLowerCase()}</option>)}
        </select>
      </label>

      <label className="filter-field">
        <span>Sort</span>
        <select
          onChange={(event) => onChange({ ...filters, sort: event.target.value as SortFilter })}
          value={filters.sort}
        >
          <option value="CLOSE_TIME">Close time</option>
          <option value="OBSERVED_TIME">Observed time</option>
        </select>
      </label>

      <button className="filter-reset" onClick={onReset} type="button">Reset filters</button>
    </form>
  );
}
```

- [ ] **Step 4: Write the mobile dialog behavior test**

Create `apps/web/src/features/radar/filter-sheet.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { beforeAll, describe, expect, it, vi } from "vitest";
import { DEFAULT_RADAR_FILTERS } from "./radar-model";
import { FilterSheet } from "./filter-sheet";

beforeAll(() => {
  HTMLDialogElement.prototype.showModal = vi.fn(function showModal(this: HTMLDialogElement) {
    this.setAttribute("open", "");
  });
  HTMLDialogElement.prototype.close = vi.fn(function close(this: HTMLDialogElement) {
    this.removeAttribute("open");
    this.dispatchEvent(new Event("close"));
  });
});

describe("FilterSheet", () => {
  it("opens modally, closes on Escape, and restores trigger focus", async () => {
    const user = userEvent.setup();
    render(
      <FilterSheet
        filters={DEFAULT_RADAR_FILTERS}
        onChange={vi.fn()}
        onReset={vi.fn()}
      />,
    );

    const trigger = screen.getByRole("button", { name: "Filters" });
    await user.click(trigger);
    expect(screen.getByRole("dialog", { name: "Market filters" })).toHaveAttribute("open");
    expect(screen.getByRole("radio", { name: "All" })).toHaveFocus();

    await user.keyboard("{Escape}");
    expect(trigger).toHaveFocus();
  });
});
```

- [ ] **Step 5: Implement the native modal bottom sheet**

Create `apps/web/src/features/radar/filter-sheet.tsx`:

```tsx
"use client";

import { useEffect, useRef, useState } from "react";
import { Button } from "@/components/ui/button";
import { FilterControls } from "./filter-controls";
import type { RadarFilters } from "./radar-model";

type FilterSheetProps = Readonly<{
  filters: RadarFilters;
  onChange: (filters: RadarFilters) => void;
  onReset: () => void;
}>;

export function FilterSheet(props: FilterSheetProps) {
  const [open, setOpen] = useState(false);
  const dialogRef = useRef<HTMLDialogElement>(null);
  const triggerRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog) return;
    if (open && !dialog.open) {
      dialog.showModal();
      dialog.querySelector<HTMLInputElement>("input")?.focus();
    }
    if (!open && dialog.open) dialog.close();
  }, [open]);

  return (
    <div className="filter-sheet-control">
      <Button ref={triggerRef} onClick={() => setOpen(true)} variant="outline">Filters</Button>
      <dialog
        aria-labelledby="filter-sheet-title"
        className="filter-sheet"
        onCancel={(event) => {
          event.preventDefault();
          setOpen(false);
        }}
        onClose={() => {
          setOpen(false);
          triggerRef.current?.focus();
        }}
        onKeyDown={(event) => {
          if (event.key === "Escape") {
            event.preventDefault();
            setOpen(false);
          }
        }}
        ref={dialogRef}
      >
        <div className="filter-sheet-header">
          <h2 id="filter-sheet-title">Market filters</h2>
          <Button aria-label="Close filters" onClick={() => setOpen(false)} variant="ghost">Close</Button>
        </div>
        <FilterControls {...props} />
      </dialog>
    </div>
  );
}
```

Before this file is implemented, replace `apps/web/src/components/ui/button.tsx` with the ref-forwarding equivalent:

```tsx
import { forwardRef, type ButtonHTMLAttributes } from "react";
import { cn } from "@/lib/utils";

type ButtonVariant = "primary" | "secondary" | "outline" | "ghost";

type ButtonProps = ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: ButtonVariant;
};

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(function Button(
  { className, variant = "primary", type = "button", ...props },
  ref,
) {
  return <button className={cn("button", `button-${variant}`, className)} ref={ref} type={type} {...props} />;
});
```

Add this test to `ui.test.tsx`:

```tsx
it("forwards button refs for focus restoration", () => {
  const ref = createRef<HTMLButtonElement>();
  render(<Button ref={ref}>Filters</Button>);
  ref.current?.focus();
  expect(screen.getByRole("button", { name: "Filters" })).toHaveFocus();
});
```

Import `createRef` from React and `Button` from `./button` in that test file.

- [ ] **Step 6: Add filter layout CSS**

Create `apps/web/src/features/radar/radar.css` with these initial filter rules; later tasks append radar row rules:

```css
.radar-layout {
  display: grid;
  grid-template-columns: 200px minmax(0, 1fr);
  gap: var(--space-10);
}

.filter-rail {
  min-width: 0;
  padding-top: var(--space-2);
  border-right: 1px solid var(--border);
  background: var(--sidebar);
}

.filter-form {
  display: grid;
  gap: var(--space-6);
  padding-right: var(--space-6);
}

.filter-group {
  min-width: 0;
  margin: 0;
  padding: 0;
  border: 0;
}

.filter-group legend,
.filter-field > span {
  display: block;
  margin-bottom: var(--space-2);
  color: var(--muted-foreground);
  font: 500 14px/20px var(--font-sans);
}

.asset-options {
  display: grid;
  gap: var(--space-1);
}

.asset-option {
  position: relative;
  display: block;
}

.asset-option input {
  position: absolute;
  opacity: 0;
}

.asset-option span {
  display: block;
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-md);
  font: 400 14px/20px var(--font-sans);
}

.asset-option input:checked + span {
  background: var(--accent);
  color: var(--accent-foreground);
  font-weight: 500;
}

.asset-option input:focus-visible + span {
  outline: 2px solid var(--ring);
  outline-offset: 2px;
}

.filter-field select {
  width: 100%;
  min-height: 40px;
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--input);
  border-radius: var(--radius-md);
  background: var(--background);
  color: var(--foreground);
  font: 400 14px/20px var(--font-sans);
}

.filter-reset {
  width: fit-content;
  padding: 0;
  border: 0;
  background: transparent;
  color: var(--muted-foreground);
  font: 500 14px/20px var(--font-sans);
  text-decoration: underline;
  text-underline-offset: 4px;
  cursor: pointer;
}

.filter-sheet-control {
  display: none;
}

.filter-sheet {
  width: min(100%, 560px);
  max-width: none;
  margin: auto 0 0;
  padding: var(--space-6);
  border: 1px solid var(--border);
  border-radius: var(--radius-xl) var(--radius-xl) 0 0;
  background: var(--popover);
  color: var(--popover-foreground);
}

.filter-sheet::backdrop {
  background: rgba(9, 9, 11, 0.64);
}

.filter-sheet-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
  margin-bottom: var(--space-6);
}

@media (max-width: 1199px) {
  .radar-layout { display: block; }
  .filter-rail { display: none; }
  .filter-sheet-control { display: block; }
  .filter-sheet .filter-form { padding-right: 0; }
}
```

Import `@/features/radar/radar.css` after `globals.css` in `layout.tsx`.

- [ ] **Step 7: Run focused filter tests**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/filter-controls.test.tsx src/features/radar/filter-sheet.test.tsx src/components/ui/ui.test.tsx
```

Expected: all focused tests pass; native `showModal()` supplies browser focus containment, Escape closes the dialog, and the trigger regains focus.

- [ ] **Step 8: Proposed application commit**

```bash
git add apps/web/src/features/radar/filter-controls.tsx apps/web/src/features/radar/filter-controls.test.tsx apps/web/src/features/radar/filter-sheet.tsx apps/web/src/features/radar/filter-sheet.test.tsx apps/web/src/features/radar/radar.css apps/web/src/components/ui/button.tsx apps/web/src/components/ui/ui.test.tsx apps/web/src/app/layout.tsx
git commit -m "feat: add responsive market filters"
```

## Task 5: Replace the fixture cards with one semantic market list

**Files:**

- Create: `apps/web/src/features/radar/market-list.tsx`
- Create: `apps/web/src/features/radar/market-list.test.tsx`
- Modify: `apps/web/src/features/radar/radar.css`
- Delete: `apps/web/src/features/foundation/market-fixture.tsx`
- Delete: `apps/web/src/features/foundation/market-fixture.test.tsx`

- [ ] **Step 1: Write semantic and typed-value tests**

Create `apps/web/src/features/radar/market-list.test.tsx`:

```tsx
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it, vi } from "vitest";
import { marketRadarFixture } from "@/lib/fixtures/markets";
import { MarketList } from "./market-list";

describe("MarketList", () => {
  it("renders one semantic list and keeps decimal prices as strings", () => {
    render(<MarketList markets={marketRadarFixture.data} onReset={vi.fn()} />);
    const list = screen.getByRole("list", { name: "Market radar results" });
    const rows = within(list).getAllByRole("listitem");
    expect(rows).toHaveLength(1);
    expect(within(rows[0]).getByText("0.58")).toBeInTheDocument();
    expect(within(rows[0]).getByText("0.42")).toBeInTheDocument();
    expect(within(rows[0]).getByText("OPEN")).toBeInTheDocument();
    expect(within(rows[0]).getByText("FRESH")).toBeInTheDocument();
  });

  it("explains filtered-empty results and resets them", async () => {
    const user = userEvent.setup();
    const onReset = vi.fn();
    render(<MarketList markets={[]} onReset={onReset} />);
    expect(screen.getByRole("heading", { name: "No markets match these filters." })).toBeInTheDocument();
    await user.click(screen.getByRole("button", { name: "Reset filters" }));
    expect(onReset).toHaveBeenCalledOnce();
  });
});
```

- [ ] **Step 2: Run the test and verify RED**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/market-list.test.tsx
```

Expected: FAIL because `MarketList` does not exist.

- [ ] **Step 3: Implement the market list, row, and typed note**

Create `apps/web/src/features/radar/market-list.tsx`:

```tsx
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import type { Market } from "@/lib/api/types";
import { formatUtcDateTime } from "@/lib/utils";

function MarketRow({ market }: Readonly<{ market: Market }>) {
  const [up, down] = market.outcomes;
  return (
    <li className="market-row">
      <div className="market-question">
        <span className="market-id">{market.marketId}</span>
        <h3>{market.question}</h3>
      </div>
      <div aria-label="Market outcome prices" className="market-outcomes">
        <div><span>{up?.label ?? "UP"}</span><strong>{up?.price ?? "—"}</strong></div>
        <div><span>{down?.label ?? "DOWN"}</span><strong>{down?.price ?? "—"}</strong></div>
      </div>
      <dl className="market-facts">
        <div><dt>Status</dt><dd>{market.status}</dd></div>
        <div><dt>Freshness</dt><dd><Badge tone={market.freshness === "FRESH" ? "success" : "warning"}>{market.freshness}</Badge></dd></div>
        <div><dt>Close time</dt><dd>{formatUtcDateTime(market.closeAt)}</dd></div>
        <div><dt>Observed</dt><dd>{formatUtcDateTime(market.observedAt)}</dd></div>
      </dl>
      <Button disabled title="Case files arrive in Slice 1" variant="secondary">Read case file</Button>
    </li>
  );
}

export function MarketList({ markets, onReset }: Readonly<{ markets: readonly Market[]; onReset: () => void }>) {
  if (markets.length === 0) {
    return (
      <section className="filtered-empty" aria-labelledby="filtered-empty-heading">
        <p className="eyebrow">No matching markets</p>
        <h3 id="filtered-empty-heading">No markets match these filters.</h3>
        <p>Reset the asset, status, and sort controls to return to the Slice 0 market boundary.</p>
        <Button onClick={onReset} variant="outline">Reset filters</Button>
      </section>
    );
  }

  return (
    <ol aria-label="Market radar results" className="market-list">
      {markets.map((market) => <MarketRow key={market.marketId} market={market} />)}
    </ol>
  );
}

export function TypedBoundaryNote() {
  return (
    <aside className="typed-boundary-note" aria-label="Typed API boundary">
      <p className="eyebrow">Boundary status</p>
      <p><strong>Read path is typed.</strong> The UI keeps decimal prices as strings and treats freshness/status as API-owned state. Wallet, thesis, and trade mutations remain intentionally outside this foundation slice.</p>
    </aside>
  );
}
```

- [ ] **Step 4: Append the dense semantic-list CSS**

Append to `radar.css`:

```css
.market-list {
  margin: 0;
  padding: 0;
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  list-style: none;
  overflow: hidden;
}

.market-row {
  display: grid;
  grid-template-columns: minmax(220px, 1.5fr) minmax(160px, 0.7fr) minmax(280px, 1fr) auto;
  align-items: center;
  gap: var(--space-6);
  padding: var(--space-5) var(--space-6);
  background: var(--card);
}

.market-row + .market-row { border-top: 1px solid var(--border); }
.market-question h3 { margin: var(--space-1) 0 0; font: 500 18px/28px var(--font-sans); }
.market-id { color: var(--muted-foreground); font: 400 12px/16px var(--font-mono); }
.market-outcomes { display: grid; grid-template-columns: repeat(2, 1fr); gap: var(--space-2); }
.market-outcomes div { padding: var(--space-2) var(--space-3); border: 1px solid var(--border); border-radius: var(--radius-md); }
.market-outcomes span { display: block; color: var(--muted-foreground); font: 500 12px/16px var(--font-sans); }
.market-outcomes strong { display: block; margin-top: var(--space-1); font: 400 16px/24px var(--font-mono); }
.market-facts { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: var(--space-3) var(--space-4); margin: 0; }
.market-facts dt { color: var(--muted-foreground); font: 400 12px/16px var(--font-sans); }
.market-facts dd { margin: var(--space-1) 0 0; font: 400 12px/16px var(--font-mono); }
.typed-boundary-note { padding: var(--space-5) var(--space-6); border: 1px dashed var(--border); border-radius: var(--radius-lg); }
.typed-boundary-note p:last-child { max-width: 760px; margin: 0; color: var(--muted-foreground); font: 400 14px/20px var(--font-sans); }
.filtered-empty { padding: var(--space-8); border: 1px solid var(--border); border-radius: var(--radius-lg); }
.filtered-empty h3 { margin: 0 0 var(--space-2); font: 500 18px/28px var(--font-sans); }
.filtered-empty p:not(.eyebrow) { max-width: 540px; color: var(--muted-foreground); font: 400 14px/20px var(--font-sans); }

@media (max-width: 1199px) {
  .market-row { grid-template-columns: minmax(0, 1fr) minmax(160px, 220px); }
  .market-facts { grid-column: 1 / -1; }
  .market-row > .button { justify-self: start; }
}

@media (max-width: 767px) {
  .market-row { grid-template-columns: 1fr; padding: var(--space-4); }
  .market-facts { grid-column: auto; grid-template-columns: 1fr; }
  .market-row > .button { width: 100%; }
  .filtered-empty { padding: var(--space-6); }
}
```

- [ ] **Step 5: Run list tests, then remove superseded fixture components**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/market-list.test.tsx
```

Expected: PASS.

Then delete the two `features/foundation/market-fixture` files. Do not delete `lib/fixtures/markets.ts`; it remains the validated Slice 0 data source.

- [ ] **Step 6: Proposed application commit**

```bash
git add apps/web/src/features/radar/market-list.tsx apps/web/src/features/radar/market-list.test.tsx apps/web/src/features/radar/radar.css apps/web/src/features/foundation/market-fixture.tsx apps/web/src/features/foundation/market-fixture.test.tsx
git commit -m "feat: render radar markets as a semantic list"
```

## Task 6: Compose `RadarSurface` with shared filters and resilient refresh

**Files:**

- Create: `apps/web/src/features/radar/radar-surface.tsx`
- Create: `apps/web/src/features/radar/radar-surface.test.tsx`
- Modify: `apps/web/src/features/radar/radar.css`

- [ ] **Step 1: Write the integration tests before the surface**

Create `apps/web/src/features/radar/radar-surface.test.tsx`:

```tsx
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { afterEach, describe, expect, it, vi } from "vitest";
import { marketRadarFixture } from "@/lib/fixtures/markets";
import { RadarSurface } from "./radar-surface";
import * as api from "@/lib/api/client";

describe("RadarSurface", () => {
  afterEach(() => vi.restoreAllMocks());

  it("shares filter state, renders the approved heading, and resets results", async () => {
    const user = userEvent.setup();
    render(<RadarSurface initialResponse={marketRadarFixture} />);

    expect(screen.getByRole("heading", { name: "A verifiable starting point." })).toBeInTheDocument();
    expect(document.querySelectorAll('form[aria-label="Market filters"]')).toHaveLength(2);
    await user.click(screen.getAllByRole("radio", { name: "ETH" })[0]);
    expect(screen.getByRole("heading", { name: "No markets match these filters." })).toBeInTheDocument();
    const resetButtons = screen.getAllByRole("button", { name: "Reset filters" });
    await user.click(resetButtons[resetButtons.length - 1]);
    expect(screen.getByRole("list", { name: "Market radar results" })).toBeInTheDocument();
  });

  it("replaces data after refresh without mutating the initial response", async () => {
    const user = userEvent.setup();
    const replacement = {
      ...marketRadarFixture,
      data: [{ ...marketRadarFixture.data[0], marketId: "replacement", question: "Will BTC close above $70,000?" }],
    };
    const initialSnapshot = structuredClone(marketRadarFixture);
    vi.spyOn(api, "listMarkets").mockResolvedValue(replacement);

    render(<RadarSurface initialResponse={marketRadarFixture} />);
    await user.click(screen.getByRole("button", { name: "Refresh markets" }));

    expect(await screen.findByText("Will BTC close above $70,000?")).toBeInTheDocument();
    expect(marketRadarFixture).toEqual(initialSnapshot);
  });

  it("preserves prior data and announces a safe refresh error", async () => {
    const user = userEvent.setup();
    vi.spyOn(api, "listMarkets").mockRejectedValue(new Error("private upstream detail"));

    render(<RadarSurface initialResponse={marketRadarFixture} />);
    await user.click(screen.getByRole("button", { name: "Refresh markets" }));

    const alert = await screen.findByRole("status");
    expect(alert).toHaveTextContent("Markets could not be refreshed. Showing the last successful response.");
    expect(screen.getByText(marketRadarFixture.data[0].question)).toBeInTheDocument();
    expect(within(screen.getByRole("list", { name: "Market radar results" })).getAllByRole("listitem")).toHaveLength(1);
  });
});
```

- [ ] **Step 2: Run the integration test and verify RED**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/radar-surface.test.tsx
```

Expected: FAIL because `RadarSurface` does not exist.

- [ ] **Step 3: Implement the interactive surface**

Create `apps/web/src/features/radar/radar-surface.tsx`:

```tsx
"use client";

import { useMemo, useState } from "react";
import { Button } from "@/components/ui/button";
import { listMarkets } from "@/lib/api/client";
import type { MarketListResponse } from "@/lib/api/types";
import { formatUtcDateTime, latestObservedAt } from "@/lib/utils";
import { FilterControls } from "./filter-controls";
import { FilterSheet } from "./filter-sheet";
import { MarketList, TypedBoundaryNote } from "./market-list";
import { DEFAULT_RADAR_FILTERS, selectRadarMarkets, type RadarFilters } from "./radar-model";

export function RadarSurface({ initialResponse }: Readonly<{ initialResponse: MarketListResponse }>) {
  const [response, setResponse] = useState<MarketListResponse>(initialResponse);
  const [filters, setFilters] = useState<RadarFilters>(DEFAULT_RADAR_FILTERS);
  const [refreshing, setRefreshing] = useState(false);
  const [refreshError, setRefreshError] = useState("");
  const markets = useMemo(() => selectRadarMarkets(response.data, filters), [response.data, filters]);
  const observedAt = latestObservedAt(response.data);

  function resetFilters() {
    setFilters(DEFAULT_RADAR_FILTERS);
  }

  async function refreshMarkets() {
    setRefreshing(true);
    setRefreshError("");
    try {
      const nextResponse = await listMarkets({ chainId: 50312, status: "OPEN", limit: 20 });
      setResponse({ ...nextResponse, data: [...nextResponse.data], meta: { ...nextResponse.meta } });
    } catch {
      setRefreshError("Markets could not be refreshed. Showing the last successful response.");
    } finally {
      setRefreshing(false);
    }
  }

  const filterProps = { filters, onChange: setFilters, onReset: resetFilters };

  return (
    <div className="radar-layout">
      <aside className="filter-rail">
        <FilterControls {...filterProps} />
      </aside>

      <div className="radar-main">
        <section className="radar-intro" aria-labelledby="radar-page-heading">
          <p className="eyebrow">Oraclelane / foundation</p>
          <h1 id="radar-page-heading">A verifiable starting point.</h1>
          <p className="lede">The application shell is ready for the market lifecycle: clear facts, explicit state, and a typed contract boundary from the first request.</p>
        </section>

        <section className="radar-panel" aria-labelledby="market-radar-heading">
          <div className="radar-header">
            <div>
              <p className="eyebrow">Live boundary preview</p>
              <h2 id="market-radar-heading">Market radar</h2>
            </div>
            <div className="radar-header-actions">
              <span className="radar-freshness">{observedAt ? `Fresh as of ${formatUtcDateTime(observedAt)}` : "No observations"}</span>
              <FilterSheet {...filterProps} />
              <Button
                aria-label="Refresh markets"
                disabled={refreshing}
                onClick={refreshMarkets}
                variant="outline"
              >
                {refreshing ? "Refreshing…" : "Refresh"}
              </Button>
            </div>
          </div>

          <p aria-live="polite" className="refresh-message" role="status">{refreshError}</p>
          <MarketList markets={markets} onReset={resetFilters} />
        </section>

        <TypedBoundaryNote />
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Finish the approved composition CSS**

Append to `radar.css`:

```css
.radar-main { min-width: 0; display: grid; gap: var(--space-9); }
.radar-intro { max-width: 760px; }
.radar-intro h1 { margin: 0 0 var(--space-4); font: 600 30px/36px var(--font-sans); letter-spacing: -0.02em; }
.radar-intro .lede { margin: 0; color: var(--muted-foreground); font: 400 16px/24px var(--font-sans); }
.radar-panel { min-width: 0; }
.radar-header { display: flex; align-items: flex-end; justify-content: space-between; gap: var(--space-6); margin-bottom: var(--space-4); }
.radar-header h2 { margin: 0; font: 600 24px/32px var(--font-sans); }
.radar-header-actions { display: flex; align-items: center; gap: var(--space-3); }
.radar-freshness { color: var(--muted-foreground); font: 400 12px/16px var(--font-mono); }
.refresh-message { min-height: 20px; margin: 0 0 var(--space-2); color: var(--destructive); font: 400 14px/20px var(--font-sans); }

@media (max-width: 1199px) {
  .radar-main { gap: var(--space-8); }
}

@media (max-width: 767px) {
  .radar-header { align-items: flex-start; flex-direction: column; }
  .radar-header-actions { width: 100%; flex-wrap: wrap; }
  .radar-freshness { width: 100%; }
  .radar-header-actions > .button { flex: 1; }
  .filter-sheet-control { flex: 1; }
  .filter-sheet-control > .button { width: 100%; }
}
```

- [ ] **Step 5: Run the surface integration tests**

Run:

```bash
pnpm --filter @oraclelane/web test -- src/features/radar/radar-surface.test.tsx
```

Expected: all tests pass; refresh replaces a copied response, failure keeps the last successful data, and desktop/mobile controls receive the same controlled state.

- [ ] **Step 6: Proposed application commit**

```bash
git add apps/web/src/features/radar/radar-surface.tsx apps/web/src/features/radar/radar-surface.test.tsx apps/web/src/features/radar/radar.css
git commit -m "feat: compose interactive Slice 0 radar surface"
```

## Task 7: Wire the server route and matching loading geometry

**Files:**

- Modify: `apps/web/src/app/radar/page.tsx`
- Modify: `apps/web/src/app/loading.tsx`
- Modify: `apps/web/src/styles/globals.css`
- Modify: `apps/web/src/features/radar/radar.css`

- [ ] **Step 1: Replace the route composition**

Set `apps/web/src/app/radar/page.tsx` to:

```tsx
import { RadarSurface } from "@/features/radar/radar-surface";
import { listMarkets } from "@/lib/api/client";

export default async function RadarPage() {
  const response = await listMarkets({ chainId: 50312, status: "OPEN", limit: 20 });
  return <RadarSurface initialResponse={response} />;
}
```

- [ ] **Step 2: Make initial loading match the final layout**

Set `apps/web/src/app/loading.tsx` to:

```tsx
import { Skeleton } from "@/components/ui/skeleton";

export default function Loading() {
  return (
    <div className="radar-layout" aria-label="Loading market radar">
      <aside className="filter-rail loading-filter-rail">
        <Skeleton className="skeleton-filter" />
        <Skeleton className="skeleton-filter" />
        <Skeleton className="skeleton-filter" />
      </aside>
      <div className="radar-main">
        <div>
          <Skeleton className="skeleton-heading" />
          <Skeleton className="skeleton-copy" />
        </div>
        <Skeleton className="skeleton-market-list" />
        <Skeleton className="skeleton-boundary-note" />
      </div>
    </div>
  );
}
```

Use exact spacing scale in the skeleton rules:

```css
.loading-filter-rail { display: grid; align-content: start; gap: var(--space-4); }
.skeleton-filter { width: calc(100% - var(--space-6)); height: 40px; }
.skeleton-heading { width: min(100%, 520px); height: 36px; }
.skeleton-copy { width: min(100%, 640px); height: 48px; margin-top: var(--space-4); }
.skeleton-market-list { width: 100%; height: 240px; }
.skeleton-boundary-note { width: 100%; height: 112px; }
```

- [ ] **Step 3: Normalize global typography and controls**

In `globals.css`, ensure the global primitives use the Figma type ramp and no visible text is below 12px:

```css
body { font: 400 14px/20px var(--font-sans); }
h1 { font: 600 30px/36px var(--font-sans); }
h2 { font: 600 24px/32px var(--font-sans); }
.eyebrow { font: 400 12px/16px var(--font-sans); }
.button { font: 500 14px/20px var(--font-sans); }
.badge { font: 400 12px/16px var(--font-mono); }
```

Retain the existing positions, trust, error, empty, and footer layouts, but convert every gap/padding/margin to the defined spacing variables. Keep borders at 1px. Remove `color-mix`-based glows and decorative elevation.

- [ ] **Step 4: Run the full component suite**

Run:

```bash
pnpm --filter @oraclelane/web test
```

Expected: all tests pass with no imports remaining from `features/foundation/market-fixture`.

Check dead imports:

```bash
rg "features/foundation|MarketFixtureCard|FoundationIntro|EmptyFixtureState|workspace-rail|Workspace navigation" apps/web/src
```

Expected: no output.

- [ ] **Step 5: Proposed application commit**

```bash
git add apps/web/src/app/radar/page.tsx apps/web/src/app/loading.tsx apps/web/src/styles/globals.css apps/web/src/features/radar/radar.css
git commit -m "feat: wire Figma-aligned radar route and loading state"
```

## Task 8: Verify coverage, build integrity, responsive behavior, and visual fidelity

**Files:**

- Modify only files implicated by verification failures.

- [ ] **Step 1: Regenerate and verify the typed boundary**

Run from `/Users/mac/developer/oraclelane`:

```bash
pnpm generate:api
pnpm lint
pnpm typecheck
pnpm test
pnpm --filter @oraclelane/web test:coverage
pnpm build
```

Expected:

- OpenAPI generation succeeds without an unexpected diff.
- ESLint reports zero errors.
- TypeScript reports zero errors.
- Vitest passes all tests.
- Lines, functions, branches, and statements each meet the configured 80% threshold.
- Next.js production build succeeds for `/radar`, `/positions`, and `/help/trust`.

- [ ] **Step 2: Check the exact implementation constraints statically**

Run:

```bash
rg -n "radial-gradient|linear-gradient\(|box-shadow|font-size: (9|10|11)px|workspace-rail|Workspace navigation" apps/web/src
rg -n -- "--background: #ffffff|--background: #09090b|--sidebar-primary: #1447e6|--radius-sm: 3.2px|--radius-xl: 11.2px" apps/web/src/styles/foundation.css
```

Expected:

- First command returns only the skeleton shimmer `linear-gradient` rule; it returns no decorative gradient, shadow, sub-12px text, or workspace rail.
- Second command returns all required exact foundation values.

- [ ] **Step 3: Start the app for visual acceptance**

Run:

```bash
pnpm dev
```

Open `http://localhost:3000/radar`. If port 3000 is occupied, use the port printed by Next.js. If the execution sandbox rejects local listening with `EPERM`, report that limitation and perform the static/build checks above; do not claim browser verification.

- [ ] **Step 4: Desktop acceptance at 1440 × 1000**

Verify against the approved composition reference:

- Header is the only global navigation surface.
- Header contains `ORACLEANE`, the three routes, `Shannon testnet`, theme control, and wallet control.
- Filter rail contains filters only and begins near the body top.
- Heading, radar header, one semantic market list, and typed note align in one restrained content column.
- Main background is true `#ffffff` in light mode.
- No gradient, glow, decorative shadow, duplicated filter, duplicated navigation, or generic BTC icon is present.
- Market values and timestamps come from typed data.

- [ ] **Step 5: Theme acceptance**

Toggle light to dark and reload each state:

- Stored theme survives reload.
- No light/dark flash occurs before hydration.
- Light and dark token values match `foundation.css`.
- Focus rings, semantic status text, border contrast, and disabled case-file action remain legible.

- [ ] **Step 6: Tablet and mobile acceptance**

Check 1024 × 900 and 390 × 844:

- Persistent filter rail is hidden below 1200px.
- One `Filters` control opens the native modal sheet.
- Sheet receives focus, Escape closes it, and focus returns to the trigger.
- Market facts wrap but remain visible.
- Question and UP/DOWN prices appear before secondary facts.
- Primary actions expand where needed.
- `document.documentElement.scrollWidth === document.documentElement.clientWidth`.

- [ ] **Step 7: Interaction and failure acceptance**

Verify:

- BTC retains the fixture; ETH shows the filtered-empty explanation.
- Status and sort selections alter real controlled state.
- Reset returns `All`, `Open`, and `Close time` and restores the fixture.
- Refresh disables only itself and preserves the existing market row.
- A mocked refresh success replaces the response.
- A mocked refresh error keeps the previous row and exposes the safe live-region message.
- `Read case file` remains disabled with the Slice 1 explanation.

- [ ] **Step 8: Inspect runtime quality**

Confirm:

- No relevant browser console errors or hydration warnings.
- No Next.js framework error overlay.
- No clipped content or page-level horizontal overflow.
- Keyboard focus order reaches wordmark, global navigation, header controls, filter controls, refresh, results, and footer logically.

- [ ] **Step 9: Final proposed application commit**

If verification required fixes and the workspace is a Git worktree:

```bash
git add apps/web
git commit -m "fix: close Slice 0 visual acceptance gaps"
```

If no fixes were needed, do not create an empty commit.

## Execution completion report

The implementer must report:

- Files created, modified, and deleted.
- Exact verification commands and their pass/fail results.
- Coverage percentages.
- Whether desktop, light, dark, tablet, and mobile were actually inspected.
- Any unverified visual checks caused by local-server restrictions.
- Whether application commits were created or skipped because the parent workspace is not a Git worktree.
- Confirmation that `architecture/docs/contracts/oraclelane.openapi.yaml` was not overwritten or committed as part of this frontend task unless generation produced an intentional, reviewed contract change.
