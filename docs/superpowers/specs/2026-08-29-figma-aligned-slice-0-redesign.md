# Figma-aligned Slice 0 redesign

Status: approved for implementation planning
Date: 2026-08-29
Figma source: [Oraclelane, node 4:2](https://www.figma.com/design/JgdpTiCVtDE6KsYB0BjENy/Oraclelane?node-id=4-2&p=f)

## Objective

Rebuild the Slice 0 Market Radar surface as a restrained decision terminal while preserving its contract-first behavior. Figma is the source of truth for colors, typography, spacing, and radii. The approved screen composition is the source of truth for layout.

The redesign does not add wallet execution, AI thesis generation, case-file navigation, or live backend filtering.

## Approved composition

### Global header

The header is the only global navigation surface. It contains:

- `ORACLEANE`
- `Market radar`
- `Positions`
- `Trust model`
- `Shannon testnet`
- theme toggle
- `Connect wallet`

`Market radar` has the selected state on `/radar`. Workspace navigation must not appear in the sidebar.

### Filter rail

The desktop sidebar contains filters only:

- Asset: `All`, `BTC`, `ETH`
- Status: `Open`
- Sort: `Close time`
- `Reset filters`

The rail begins near the top of the application body. It must not repeat header navigation or duplicate controls from the main panel.

### Radar content

The main area contains:

1. Page heading `A verifiable starting point.`
2. Existing Slice 0 supporting copy.
3. A Market Radar panel header with `Market radar`, a UTC freshness timestamp, and `Refresh`.
4. Market rows that expose the exact question, UP/DOWN prices, status, freshness, close time, observed time, and `Read case file` action.
5. The existing typed-boundary note led by `Read path is typed.`

The primary desktop representation is a bordered market table/list. It must not become a card grid, bento layout, or marketing hero.

## Figma foundation

### Color tokens

Light mode:

| Token | Value |
|---|---|
| background | `#ffffff` |
| foreground | `#09090b` |
| card | `#ffffff` |
| popover | `#ffffff` |
| primary | `#18181b` |
| secondary | `#f4f4f5` |
| muted | `#f4f4f5` |
| accent | `#f4f4f5` |
| destructive | `#e7000b` |
| border | `#e4e4e7` |
| input | `#e4e4e7` |
| ring | `#9f9fa9` |
| chart-1 | `#d4d4d8` |
| chart-2 | `#71717b` |
| chart-3 | `#52525c` |
| chart-4 | `#3f3f46` |
| chart-5 | `#27272a` |
| sidebar | `#fafafa` |
| sidebar-primary | `#18181b` |

Dark mode:

| Token | Value |
|---|---|
| background | `#09090b` |
| foreground | `#fafafa` |
| card | `#18181b` |
| popover | `#18181b` |
| primary | `#e4e4e7` |
| secondary | `#27272a` |
| muted | `#27272a` |
| accent | `#27272a` |
| destructive | `#ff6467` |
| border | `rgba(255,255,255,0.10)` |
| input | `rgba(255,255,255,0.15)` |
| ring | `#71717b` |
| chart-1 | `#d4d4d8` |
| chart-2 | `#71717b` |
| chart-3 | `#52525c` |
| chart-4 | `#3f3f46` |
| chart-5 | `#27272a` |
| sidebar | `#18181b` |
| sidebar-primary | `#1447e6` |

The main light background is true white. Do not replace it with cream or a tinted neutral. Gradients, glass effects, glows, and decorative shadows are excluded.

Outcome and freshness colors may use restrained semantic green and red where needed for comprehension and status, subject to accessible contrast in both themes.

### Typography

Use Inter for product text and JetBrains Mono for prices, timestamps, statuses, chain values, and other machine-readable data.

| Style | Family | Weight | Size / line height |
|---|---|---:|---:|
| Display/2xl | Inter | 600 | `30px / 36px` |
| Heading/xl | Inter | 600 | `24px / 32px` |
| Heading/lg | Inter | 600 | `20px / 28px` |
| Heading/md | Inter | 500 | `18px / 28px` |
| Body/base | Inter | 400 | `16px / 24px` |
| Body/sm | Inter | 400 | `14px / 20px` |
| Caption/xs | Inter | 400 | `12px / 16px` |
| Label/sm | Inter | 500 | `14px / 20px` |
| Mono/base | JetBrains Mono | 400 | `16px / 24px` |
| Mono/sm | JetBrains Mono | 400 | `14px / 20px` |
| Mono/xs | JetBrains Mono | 400 | `12px / 16px` |

Visible text must not be smaller than 12px. Controls receive explicit typography and never rely on browser defaults.

### Spacing and radii

Use only the Figma spacing scale for layout and component padding: `4`, `8`, `12`, `16`, `20`, `24`, `32`, `40`, `48`, and `64px`.

Use the radius scale derived from `--radius: 0.45rem`:

- small: `3.2px`
- medium: `5.2px`
- large: `7.2px`
- extra large: `11.2px`

Borders are 1px. Elevation is minimal and must not turn rows or sections into floating cards.

## Component boundaries

- `AppShell`: global header, desktop body grid, footer, and theme integration.
- `PrimaryNavigation`: route-aware header navigation only.
- `FilterRail`: desktop filter controls and reset action.
- `FilterSheet`: mobile presentation of the same filter state and controls.
- `RadarSurface`: owns interactive filter, sort, refresh, loading, and error state.
- `RadarHeader`: panel title, freshness timestamp, and refresh control.
- `MarketTable`: semantic list/table container and empty state.
- `MarketRow`: one typed market record with outcomes and operational facts.
- `TypedBoundaryNote`: existing contract-first explanation.

Shared controls continue to use the existing button, badge, separator, and skeleton primitives when their semantics match. Differences should be explicit variants rather than copied markup.

## Data and interaction flow

1. The `/radar` server route obtains the initial `MarketListResponse` through `listMarkets`.
2. It passes typed market data to `RadarSurface`.
3. Asset, status, and sort controls update local UI state without mutating the fixture.
4. Asset filtering matches the market question or future explicit asset field. `All` restores the complete response.
5. `Reset filters` restores `All`, `Open`, and `Close time`.
6. `Refresh` invokes the same typed fetcher, replaces the local response with a new immutable result, and updates the visible UTC freshness timestamp.
7. A refresh failure preserves the last successful data, exposes an inline user-readable error, and announces it through an accessible live region.

Every control must update real local UI state. Decorative or inert filter controls are not acceptable.

## Responsive behavior

### Desktop, 1200px and above

- Persistent filter rail.
- Main content uses remaining width.
- Market rows keep a dense horizontal facts layout.

### Tablet, 768px to 1199px

- Filter rail collapses into a `Filters` toolbar control.
- Market facts may wrap into two logical rows without hiding contract data.

### Mobile, below 768px

- One-column layout.
- `Filters` opens an accessible bottom sheet.
- Market question and UP/DOWN prices remain first.
- Status, freshness, close time, and observed time remain visible in the row detail.
- Primary actions use full available width when necessary.
- No horizontal page overflow is allowed.

## Theme behavior

Light and dark themes use the exact Figma token values. Theme selection remains user-controlled and persisted. The pre-hydration theme application must prevent a visible color flash and must not introduce a hydration warning.

## Accessibility

- Header and filter navigation use distinct accessible landmarks.
- Selected routes use `aria-current="page"`.
- Filter groups have visible labels and keyboard-operable controls.
- The mobile sheet traps focus while open, closes with Escape, and restores focus to its trigger.
- Refresh loading and failure states use a polite live region.
- Focus rings use the Figma `ring` token and remain visible in both themes.
- Semantic status colors never carry meaning without text.

## Error, loading, and empty states

- Initial loading uses skeleton geometry matching the final screen.
- Refresh loading disables only the refresh action and preserves existing data.
- Refresh errors appear inline within the Radar panel.
- Empty filtered results explain which filters produced no matches and provide `Reset filters`.
- Route-level error and not-found boundaries remain available.

## Testing and acceptance

### Unit and integration tests

- Exact light and dark token values are present.
- Theme selection applies and persists correctly.
- Header navigation has one active item and no sidebar duplication.
- Asset, status, sort, reset, and refresh controls update state.
- Refresh success replaces data immutably.
- Refresh failure preserves prior data and announces the error.
- Desktop filter rail and mobile filter sheet share state.
- Market rows retain decimal prices as strings.
- Keyboard navigation and mobile-sheet focus restoration work.

### Visual acceptance

- Desktop implementation matches the approved revised concept.
- Light and dark modes are checked.
- A mobile viewport below 768px is checked.
- No duplicate navigation or duplicate filters are visible.
- Figma palette, type ramp, spacing, radii, and white background are preserved.
- No clipping, overflow, framework overlay, or relevant console errors remain.

### Required verification commands

```bash
pnpm generate:api
pnpm lint
pnpm typecheck
pnpm test
pnpm --filter @oraclelane/web test:coverage
pnpm build
```

## Intentional differences from the approval image

- Dates, freshness times, and market values come from typed application data rather than rasterized concept text.
- The application does not use a generic Bitcoin icon unless an exact approved asset exists.
- The typed-boundary body copy remains the existing repository copy rather than the image generator's illustrative sentence.
- Responsive states are implemented as real components rather than inferred from the desktop bitmap.

These differences protect data truth, asset fidelity, and accessibility without changing the approved layout direction.
