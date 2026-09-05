# 20. Oraclelane design system

> **✅ Canonical token source.** This is the token set actually used by the Figma design system and generated screens. Root `DESIGN.md` and `blueprint/docs/16-ux-design-system.md` predate this build and define different (blue-primary / green-primary) palettes — both are now marked superseded and point back here. If code and this doc ever disagree, treat this file as the source of truth for tokens, per its own framing below.

Oraclelane builds on the **shadcn/ui token names**, with a palette of its own.

> **Superseded, 2026-09-05.** This file previously said Oraclelane "does not introduce a separate brand palette at this stage; product semantics are expressed through shadcn semantic tokens." That decision produced a palette in which every token had chroma 0–0.017, `--chart-1` through `--chart-5` shared one hue, and `--card` was identical to `--background` in light mode. The system had no way to say *this is important*, so every panel on a page rendered at the same weight — and the safety gate's `ALLOW` / `WARN` / `BLOCK`, which is the product's core vocabulary, had no visual expression at all. Colour is now admitted, and it carries meaning rather than brand.

Three additions, each with a rule attached:

- **`--signal`** — reserved for chain-confirmed state (fresh, observed, confirmed). It means "the chain says so", never "we thought this looked important".
- **`--verdict-allow` / `--verdict-warn` / `--verdict-block`** — the safety gate's decision. Colour is a second channel only: the badge still spells the decision and each check still carries a tick or a cross, per WCAG 2.2 AA.
- **`--surface-sunken`** — a third surface tier below `--background` (base) and `--card` (raised), so panels can differ in weight.

`--chart-1..5` are five separable hues. Their steps were chosen with a validator (lightness band, chroma floor, CVD separation, normal-vision floor, contrast against each mode's surface), not by eye. Outcomes are charted cyan and violet rather than green and red: an outcome is a direction, and colouring UP green would tell a reader the market has a good side.

The values below are the source of truth, mirrored exactly by `apps/web/src/styles/foundation.css`. That file declares raw values only; `globals.css` `@theme inline` is the sole place a token becomes a Tailwind utility, and the sole owner of the radius scale — the two files used to declare `--radius-sm|md|lg|xl` with different values, and the browser silently used one while a test asserted the other.

`apps/web/src/styles/foundation.test.ts` asserts palette **invariants** rather than exact values: both themes define the same tokens, text clears WCAG AA against its surface, raised surfaces separate from the page, and no token has two owners.

## Canonical shadcn tokens

```css
:root {
  color-scheme: light;

  /* Surfaces: base (page) < raised (panel) > sunken (data well). */
  --background: oklch(0.96 0.004 250);
  --card: oklch(1 0 0);
  --popover: oklch(1 0 0);
  --surface-sunken: oklch(0.925 0.005 250);

  --foreground: oklch(0.17 0.01 250);
  --card-foreground: oklch(0.17 0.01 250);
  --popover-foreground: oklch(0.17 0.01 250);
  --muted-foreground: oklch(0.5 0.015 250);

  --primary: oklch(0.22 0.012 250);
  --primary-foreground: oklch(0.98 0.002 250);
  --secondary: oklch(0.945 0.005 250);
  --secondary-foreground: oklch(0.22 0.012 250);
  --muted: oklch(0.945 0.005 250);
  --accent: oklch(0.945 0.005 250);
  --accent-foreground: oklch(0.22 0.012 250);

  --destructive: oklch(0.55 0.22 25);
  --border: oklch(0.89 0.006 250);
  --input: oklch(0.87 0.007 250);
  --ring: oklch(0.55 0.13 198);

  --signal: oklch(0.52 0.12 198);
  --signal-foreground: oklch(0.99 0.002 250);

  --verdict-allow: oklch(0.5 0.13 152);
  --verdict-allow-foreground: oklch(0.99 0.002 250);
  --verdict-warn: oklch(0.55 0.13 78);
  --verdict-warn-foreground: oklch(0.99 0.002 250);
  --verdict-block: oklch(0.53 0.21 25);
  --verdict-block-foreground: oklch(0.99 0.002 250);

  --chart-1: oklch(0.62 0.13 200);
  --chart-2: oklch(0.53 0.13 152);
  --chart-3: oklch(0.6 0.13 78);
  --chart-4: oklch(0.52 0.16 295);
  --chart-5: oklch(0.56 0.18 15);

  --sidebar: oklch(0.98 0.003 250);
  --sidebar-foreground: oklch(0.17 0.01 250);
  --sidebar-primary: oklch(0.22 0.012 250);
  --sidebar-primary-foreground: oklch(0.98 0.002 250);
  --sidebar-accent: oklch(0.945 0.005 250);
  --sidebar-accent-foreground: oklch(0.22 0.012 250);
  --sidebar-border: oklch(0.89 0.006 250);
  --sidebar-ring: oklch(0.55 0.13 198);

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

  --font-sans: var(--font-inter, "Inter"), ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-mono: var(--font-jetbrains-mono, "JetBrains Mono"), "SFMono-Regular", Consolas, monospace;
}

.dark {
  color-scheme: dark;

  --background: oklch(0.145 0.008 250);
  --card: oklch(0.225 0.01 250);
  --popover: oklch(0.225 0.01 250);
  --surface-sunken: oklch(0.115 0.008 250);

  --foreground: oklch(0.97 0.003 250);
  --card-foreground: oklch(0.97 0.003 250);
  --popover-foreground: oklch(0.97 0.003 250);
  --muted-foreground: oklch(0.7 0.015 250);

  --primary: oklch(0.95 0.005 250);
  --primary-foreground: oklch(0.19 0.01 250);
  --secondary: oklch(0.28 0.012 250);
  --secondary-foreground: oklch(0.97 0.003 250);
  --muted: oklch(0.28 0.012 250);
  --accent: oklch(0.28 0.012 250);
  --accent-foreground: oklch(0.97 0.003 250);

  --destructive: oklch(0.7 0.19 25);
  --border: oklch(1 0 0 / 12%);
  --input: oklch(1 0 0 / 16%);
  --ring: oklch(0.72 0.13 198);

  --signal: oklch(0.78 0.12 198);
  --signal-foreground: oklch(0.16 0.01 250);

  --verdict-allow: oklch(0.78 0.14 152);
  --verdict-allow-foreground: oklch(0.16 0.01 250);
  --verdict-warn: oklch(0.83 0.13 78);
  --verdict-warn-foreground: oklch(0.16 0.01 250);
  --verdict-block: oklch(0.7 0.19 25);
  --verdict-block-foreground: oklch(0.16 0.01 250);

  --chart-1: oklch(0.65 0.12 200);
  --chart-2: oklch(0.64 0.14 152);
  --chart-3: oklch(0.66 0.13 78);
  --chart-4: oklch(0.6 0.17 292);
  --chart-5: oklch(0.6 0.17 15);

  --sidebar: oklch(0.225 0.01 250);
  --sidebar-foreground: oklch(0.97 0.003 250);
  --sidebar-primary: oklch(0.78 0.12 198);
  --sidebar-primary-foreground: oklch(0.16 0.01 250);
  --sidebar-accent: oklch(0.28 0.012 250);
  --sidebar-accent-foreground: oklch(0.97 0.003 250);
  --sidebar-border: oklch(1 0 0 / 12%);
  --sidebar-ring: oklch(0.72 0.13 198);
}
```

## Oraclelane usage rules

| Product meaning | shadcn token/component | Rule |
|---|---|---|
| Primary navigation/action | `primary`, `Button` | Use for `Read case file`, `Review trade`, and explicit wallet actions |
| Supporting action | `secondary`, `outline`, `ghost` | Use for filters, refresh, copy, and navigation |
| Chain/evidence facts | `muted`, `muted-foreground`, `Badge` | Never style facts as an AI prediction |
| AI thesis | `card`, `accent`, `Alert` | Label as advisory; no guarantee-like visual treatment |
| Stale/uncertain | `secondary` + text/icon | Avoid inventing an orange palette; copy must explain the impact |
| Blocked/failed | `destructive`, `Alert` | Explain the deterministic check and the next action |
| Confirmed/redeemed | `primary` or `secondary` + explicit label | Do not imply profit; show chain confirmation |

Trading direction (`UP`/`DOWN`) must not be encoded by green/red alone. Use text, icon, and a neutral chart treatment; direction is a market outcome label, not a success/failure signal.

## shadcn component inventory

Use existing shadcn components and compose them before adding custom markup:

| Need | Component |
|---|---|
| Navigation | `Sidebar`, `NavigationMenu`, `Tabs`, `Breadcrumb` |
| Market display | `Card`, `Badge`, `Table`, `Chart`, `Skeleton` |
| Filters and inputs | `Select`, `ToggleGroup`, `Input`, `Slider`, `FieldGroup`/`Field` |
| Trade review | `Sheet` (desktop), `Drawer` (mobile), `AlertDialog` (final confirmation) |
| Feedback | `Alert`, `Progress`, `Spinner`, `sonner` |
| Empty/error states | `Empty`, `Alert` |
| Utility | `Tooltip`, `Popover`, `Separator`, `ScrollArea`, `CopyableValue` (thin composition) |

Required composition rules: every `Sheet`/`Drawer` has an accessible title; every `Card` uses its header/content/footer structure where applicable; loading uses `Skeleton`; status uses `Badge`; form validation uses `data-invalid` and `aria-invalid`.

## Typography and layout

Use the shadcn project font configuration with a readable sans-serif and a monospace face for addresses, IDs, amounts, and transaction hashes. Use tabular numerals for prices and timestamps. The spacing scale follows the project Tailwind/shadcn preset; do not add arbitrary spacing values in Figma or code.

## Accessibility and motion

- Target WCAG AA contrast for both themes.
- Focus order follows the trading task flow; focus enters and returns from sheets/drawers.
- Every status has text and an accessible label; color is supplementary.
- Respect `prefers-reduced-motion`.
- Use 150ms ease-out for hover/focus and 220ms ease-out for sheet/drawer transitions.
- Never use looping celebration animation for a financial state.

## Theme decision

Design both themes from the same semantic tokens, with **dark mode as the default demo presentation** because it supports the calm terminal direction and makes dense market information comfortable during judging. Light mode remains a supported accessibility and user preference, not a separate visual system.
