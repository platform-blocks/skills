---
name: plocks-charts
description: Add data visualization to a React Native or Expo app with @plocks/charts. Use when installing the charts package, choosing among its 24 chart types (line, bar, area, pie/donut, scatter, sparkline, heatmap, candlestick, sankey, and more), theming charts to match the app via ChartThemeProvider and hostThemeBridge, wiring tooltips/crosshairs/pan-zoom interactions, or rendering streaming/live data.
---

# plocks Charts

`@plocks/charts` is a React Native + Web charting library (SVG via
`react-native-svg`, animations via `react-native-reanimated`) with 24 chart
components, a shared interaction engine (tooltips, crosshair, pan/zoom), and a
theme bridge for host design systems. Docs: https://plocks.dev

- Chart catalog + core APIs: `references/api.md`
- Complete copy-paste examples: `references/patterns.md`

## Install

```bash
npm install @plocks/charts react-native-svg react-native-reanimated
# Expo: npx expo install react-native-svg react-native-reanimated
```

Peer dependencies: `react >= 18`, `react-native >= 0.73`,
`react-native-svg >= 13`, `react-native-reanimated >= 3.4`
(`react-dom` optional, web only). Reanimated needs its Babel/worklets plugin
configured in the consuming app.

## Theming: ChartThemeProvider + hostThemeBridge

Wrap the app (or any subtree) in `ChartThemeProvider`. Without it, charts use a
light default theme. To match the host app, pass `hostThemeBridge` — all fields
optional: `textPrimary`, `textSecondary`, `background`, `grid`,
`accentPalette` (string[]), `fontFamily`.

Real pattern from plocks.dev (bridging `@plocks/ui`):

```tsx
import { useTheme } from '@plocks/ui';
import { ChartThemeProvider } from '@plocks/charts';

// Categorical series palette — fixed hue order, assigned by slot, never cycled.
// Pinned as hex (not theme ramp indexes) and validated for CVD separation and
// 3:1 contrast against BOTH light and dark surfaces, so a series keeps its
// color across a theme toggle.
const CHART_SERIES_PALETTE = [
  '#3B82F6', '#16A34A', '#A855F7', '#D97706',
  '#0891B2', '#65A30D', '#6366F1', '#EF4444',
];

const ChartThemeBridge = ({ children }) => {
  const theme = useTheme();
  const hostBridge = React.useMemo(() => ({
    textPrimary: theme.text.primary,
    textSecondary: theme.text.secondary,
    background: theme.backgrounds.surface,
    grid: theme.colors.gray?.[3] ?? theme.backgrounds.border ?? '#e5e7eb',
    accentPalette: CHART_SERIES_PALETTE,
    fontFamily: theme.fontFamily,
  }), [theme]);
  return <ChartThemeProvider hostThemeBridge={hostBridge}>{children}</ChartThemeProvider>;
};
```

Notes:
- If you pass a dark `background` but no `accentPalette`, the provider
  auto-selects its built-in dark-surface palette (the light one washes out to
  ~2:1 contrast on dark). An explicit `accentPalette` always wins.
- The provider also feeds the palette into the module-level color scheme
  (`setDefaultColorScheme`), so series without explicit colors pick it up.
- `useChartTheme()` reads the merged `ChartTheme` inside any chart subtree.
- Axis ticks and value labels abbreviate large numbers by default (9000 →
  "9K"). Opt out with `<ChartThemeProvider value={{ numberFormat: 'full' }}>`,
  or override one axis with `labelFormatter`. Tooltips always show full values.

## Picking a chart type

| Intent | Use |
|---|---|
| Trend over time | `LineChart`, `AreaChart`, `SparklineChart` (inline/KPI) |
| Part-to-whole | `PieChart`, `DonutChart`, `StackedBarChart`, `StackedAreaChart`, `MarimekkoChart` |
| Category comparison | `BarChart`, `GroupedBarChart`, `RadialBarChart`, `ParetoChart` |
| Correlation / 2–3 variables | `ScatterChart`, `BubbleChart` |
| Distribution | `HistogramChart`, `ViolinChart`, `RidgeChart` |
| Matrix / intensity | `HeatmapChart` |
| Flows and relationships | `SankeyChart`, `NetworkChart`, `FunnelChart` |
| Financial OHLC | `CandlestickChart` |
| Multivariate profile | `RadarChart` |
| Mixed layers (bar + line + area) | `ComboChart` |

Full catalog with exact data-prop shapes: `references/api.md`.

## Data shapes (most common)

- XY charts (`LineChart`, `AreaChart`, `ScatterChart`): `data: ChartDataPoint[]`
  (`{ x: number; y: number; id?, label?, color?, size?, data? }`) or
  `series: [{ name, data, color?, ... }]` for multi-series.
- `BarChart`: `data: BarChartDataPoint[]` (`{ category: string; value: number; color? }`);
  multi-series via `series` + `layout: 'grouped' | 'stacked'` (+ `stackMode: '100%'`).
- `PieChart` / `DonutChart`: `data: { label: string; value: number; color? }[]`.
- `SparklineChart`: `data: number[] | { x, y }[]`.
- Everything accepts `w`/`h`, `title`/`subtitle`, spacing props
  (`m`, `mx`, `p`, ...) and accessibility props from `BaseChartProps`.

## Interactions

Per-chart props (Line/Bar/Area/Scatter and most cartesian charts):
- Tooltip: `tooltip={{ show, formatter, backgroundColor, ... }}`,
  `liveTooltip` (follow pointer), `multiTooltip` (all series at pointer x),
  `enableCrosshair`.
- Pan/zoom (Line/Scatter): `enablePanZoom`, `zoomMode: 'x' | 'y' | 'both'`,
  `minZoom`, `enableWheelZoom`, `wheelZoomStep`, `invertWheelZoom`,
  `invertPinchZoom`, `resetOnDoubleTap`, `clampToInitialDomain`,
  `onDomainChange`.
- Events: `onPress`, `onDataPointPress` (typed per chart's datum).

Multiple charts sharing one crosshair/tooltip/zoom: wrap in `ChartsProvider`
(alias `GlobalChartsRoot`), give each chart
`useOwnInteractionProvider={false}` and `suppressPopover`, and let the provider
render the single shared popover (`withPopover`, default true). Customize it by
mounting `ChartActiveTooltip` yourself (`render`, `renderEntry`,
`renderHeader`, `filterEntry`, `sortEntries`, `maxEntries`, `offset`). See
`references/patterns.md`.

## Streaming / live data

Use `useStreamingData(initialData, { maxDataPoints, updateInterval, onDataOverflow })`
— returns `{ data, addDataPoint, startStreaming, stopStreaming, clearData, isStreaming }`.
It batches high-frequency points and trims to a rolling window; feed `data`
straight into a `LineChart`/`AreaChart`. For very large static series, pass
`decimationThreshold` (LineChart) or pre-thin with `useDataDecimation`
(LTOB, preserves visual trend).

## Accessibility

Every chart accepts `accessible`, `accessibilityLabel`, `accessibilityHint`,
`accessibilityRole`, and `importantForAccessibility`. Always set an
`accessibilityLabel` that summarizes the data. `PieChart` additionally
supports `keyboardNavigation` and `ariaLabelFormatter(slice, percentage)`.

## Pitfalls

- Charts render nothing useful without explicit or inherited size — set
  `width`/`height` (or `size` on `DonutChart`) when the layout doesn't provide one.
- `accentPalette` colors are assigned by series slot in a fixed order; keep the
  order stable so series keep their identity across renders and theme switches.
- Inside `ChartsProvider`, forgetting `useOwnInteractionProvider={false}` gives
  each chart its own isolated tooltip/zoom state; forgetting `suppressPopover`
  double-renders tooltips.
- Time axes take numeric timestamps (`x: Date.now()`) with
  `xScaleType="time"`, not `Date` objects (exception: `CandlestickChart`
  accepts `x: number | Date`).
- Reanimated misconfiguration in the consumer app breaks animations; as a
  debug fallback `LineChart` supports `disableAnimations`.

## Anything this skill does not cover

This skill covers the separate `@plocks/charts` package. plocks
is much larger — 100+ core UI components, 24 chart types, and 48 documented
hooks. Do not guess an API for something outside this scope; fetch the generated
docs instead:

| What you need | Where |
| --- | --- |
| Index of every page, one line each | `https://plocks.dev/llms.txt` |
| One component or chart | `https://plocks.dev/llms/components/<Name>.md` |
| One hook | `https://plocks.dev/llms/hooks/<useName>.md` |
| Guides | `https://plocks.dev/llms/guides/{getting-started,accessibility,localization}.md` |
| Everything in one file (~1.3 MB) | `https://plocks.dev/llms-full.txt` |

`<Name>` is the exact PascalCase export name — `.../llms/components/DataTable.md`,
`.../llms/components/AreaChart.md`. Each page carries the component's full prop
table (type, required, default, description) plus runnable examples, generated
from the source, so it is authoritative where memory is not. When you are unsure
whether something exists or what it is called, read `llms.txt` first — it lists
every page with a one-line summary.

Import components from their owning package: `@plocks/ui` for core UI,
`@plocks/charts`, `@plocks/dates`, `@plocks/code`, `@plocks/media`,
`@plocks/qrcode`, and `@plocks/spotlight` for extensions. `FormLayout` is
also exported from `@plocks/ui`; the `Navigation` module uses the
`@plocks/ui/Navigation` subpath. Check the package barrel before importing.

Notably outside this skill:

- **The UI library itself** (`@plocks/ui`) — its 100+ components, theme
  system, and layout primitives are a different package. Charts are usually
  placed inside a `Card`/`Surface` from it.
- **Install and provider wiring** → the `plocks-setup` skill.
  **Bridging chart colors to the app theme** is covered above; the theme object
  itself is in `plocks-theming`. **Placing charts on a screen** →
  `plocks-layout`.
