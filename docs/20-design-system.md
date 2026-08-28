# 20. Oraclelane design system

> **✅ Canonical token source.** This is the token set actually used by the Figma design system and generated screens. Root `DESIGN.md` and `blueprint/docs/16-ux-design-system.md` predate this build and define different (blue-primary / green-primary) palettes — both are now marked superseded and point back here. If code and this doc ever disagree, treat this file as the source of truth for tokens, per its own framing below.

Oraclelane uses the **shadcn/ui neutral design system** as its visual foundation. The CSS variables below are the canonical tokens for Figma and the future frontend. Oraclelane does not introduce a separate brand palette at this stage; product semantics are expressed through shadcn semantic tokens and documented component variants.

## Canonical shadcn tokens

```css
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.141 0.005 285.823);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.141 0.005 285.823);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.141 0.005 285.823);
  --primary: oklch(0.21 0.006 285.885);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.967 0.001 286.375);
  --secondary-foreground: oklch(0.21 0.006 285.885);
  --muted: oklch(0.967 0.001 286.375);
  --muted-foreground: oklch(0.552 0.016 285.938);
  --accent: oklch(0.967 0.001 286.375);
  --accent-foreground: oklch(0.21 0.006 285.885);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.92 0.004 286.32);
  --input: oklch(0.92 0.004 286.32);
  --ring: oklch(0.705 0.015 286.067);
  --chart-1: oklch(0.871 0.006 286.286);
  --chart-2: oklch(0.552 0.016 285.938);
  --chart-3: oklch(0.442 0.017 285.786);
  --chart-4: oklch(0.37 0.013 285.805);
  --chart-5: oklch(0.274 0.006 286.033);
  --radius: 0.45rem;
  --sidebar: oklch(0.985 0 0);
  --sidebar-foreground: oklch(0.141 0.005 285.823);
  --sidebar-primary: oklch(0.21 0.006 285.885);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.967 0.001 286.375);
  --sidebar-accent-foreground: oklch(0.21 0.006 285.885);
  --sidebar-border: oklch(0.92 0.004 286.32);
  --sidebar-ring: oklch(0.705 0.015 286.067);
}

.dark {
  --background: oklch(0.141 0.005 285.823);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.21 0.006 285.885);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.21 0.006 285.885);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.92 0.004 286.32);
  --primary-foreground: oklch(0.21 0.006 285.885);
  --secondary: oklch(0.274 0.006 286.033);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.274 0.006 286.033);
  --muted-foreground: oklch(0.705 0.015 286.067);
  --accent: oklch(0.274 0.006 286.033);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.552 0.016 285.938);
  --chart-1: oklch(0.871 0.006 286.286);
  --chart-2: oklch(0.552 0.016 285.938);
  --chart-3: oklch(0.442 0.017 285.786);
  --chart-4: oklch(0.37 0.013 285.805);
  --chart-5: oklch(0.274 0.006 286.033);
  --sidebar: oklch(0.21 0.006 285.885);
  --sidebar-foreground: oklch(0.985 0 0);
  --sidebar-primary: oklch(0.488 0.243 264.376);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.274 0.006 286.033);
  --sidebar-accent-foreground: oklch(0.985 0 0);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.552 0.016 285.938);
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
