# @platform-blocks/charts — Chart catalog and core APIs

All components and types below are exported from the package root:
`import { LineChart, ChartThemeProvider, ... } from '@platform-blocks/charts';`

## Chart catalog (25 components)

| Component | Primary data prop | Description |
|---|---|---|
| `LineChart` | `data: ChartDataPoint[]` or `series: LineChartSeries[]` | Trends over a continuous x axis; smoothing, fills, crosshair, pan/zoom, annotations. |
| `AreaChart` | same as `LineChart` + `layout` | Line chart with filled areas; `layout: 'overlap' \| 'stacked' \| 'stackedPercentage'`. |
| `StackedAreaChart` | `series: LineChartSeries[]` | Multiple series stacked to show cumulative contributions. |
| `BarChart` | `data: BarChartDataPoint[]` (+ optional `series`) | Category comparison; vertical/horizontal, grouped/stacked layouts, value labels, thresholds. |
| `GroupedBarChart` | `series: StackedBarSeries[]` | Side-by-side bars per category for series comparison. |
| `StackedBarChart` | `series: StackedBarSeries[]` | Part-to-whole per category with stacked segments. |
| `PieChart` | `data: PieChartDataPoint[]` | Proportions as slices; labels with leader lines, gradients, multi-ring `layers`. |
| `DonutChart` | `data: DonutChartDataPoint[]` or `rings: DonutChartRing[]` | Pie with center content; multi-ring, center label/value formatters, `isolateOnClick`. |
| `ScatterChart` | `data: ChartDataPoint[]` or `series: ScatterSeries[]` | Correlation of two variables; trendlines, quadrant overlays, pan/zoom. |
| `BubbleChart` | `data: T[]` + `dataKey` mapping | Scatter where bubble area encodes a third dimension; generic record data with key mapping. |
| `SparklineChart` | `data: number[] \| SparklinePoint[]` | Minimal inline trend line for KPI tiles; last-value bubble, extrema markers, thresholds, bands. |
| `HistogramChart` | `data: number[]` | Distribution of raw values split into bins; emits `HistogramBinSummary` via `onBinFocus`/`onBinBlur`. |
| `CandlestickChart` | `series: CandlestickSeries[]` | Financial OHLC candles (`{ x, open, high, low, close, volume? }`); moving-average overlays. |
| `ComboChart` | `layers: ComboChartLayer[]` | Mixed layers on shared axes; layer `type: 'bar' \| 'line' \| 'area' \| 'histogram' \| 'density'`, per-layer `targetAxis: 'left' \| 'right'`. |
| `ParetoChart` | `data: ParetoChartDatum[]` | Bars sorted by impact plus cumulative-percentage line (built on ComboChart). |
| `RadarChart` | `series: RadarChartSeries[]` (`data: RadarAxisPoint[]` per series, `{ axis, value }`) | Multivariate profiles on radial axes. |
| `RadialBarChart` | `data: RadialBarDatum[]` (`{ value, max?, label?, color? }`) | Values as concentric circular bars. |
| `GaugeChart` | `value: number` (+ `min?`, `max?`) | Bounded progress on a semicircular gauge; ranges, ticks, markers, animated needle, `onValueChange`. |
| `HeatmapChart` | `data: HeatmapCell[] \| HeatmapMatrixInput` | Color-coded matrix; cells `{ x, y, value }` or `{ rows, cols, values[][] }`. |
| `FunnelChart` | `series: FunnelChartSeries \| FunnelChartSeries[]` (`steps: { label, value }[]`) | Progressive stage reduction (conversion pipelines). |
| `RidgeChart` | `series: DensitySeries[]` (`values: number[]` per series) | Layered density curves (joyplot) comparing distributions. |
| `ViolinChart` | `series: ViolinDensitySeries[]` | Distribution with kernel density plus statistic markers (median/mean/quartiles/whiskers). |
| `SankeyChart` | `nodes: SankeyNode[]`, `links: SankeyLink[]` | Flow volumes between nodes; links `{ source, target, value }`. |
| `NetworkChart` | `nodes: NetworkNode[]`, `links: NetworkLink[]` | Graph relationships; force/coordinate/circular/radial layouts, `onNodeFocus`/`onLinkFocus`. |
| `MarimekkoChart` | `data: MarimekkoCategory[]` | Mosaic of variable-width columns: categorical mix vs. overall weight. |

## Common data shapes (verbatim from source)

```ts
// XY charts (Line/Area/Scatter, and Sparkline points)
interface ChartDataPoint {
  id?: string | number;
  x: number;
  y: number;
  label?: string;
  color?: string;
  size?: number;
  data?: any;            // custom payload surfaced in interaction events
}

interface LineChartSeries {
  id?: string | number;
  name?: string;
  data: ChartDataPoint[];
  color?: string;
  thickness?: number;    // alias: lineThickness
  style?: 'solid' | 'dashed' | 'dotted';  // alias: lineStyle
  showPoints?: boolean;
  pointSize?: number;
  pointColor?: string;
  visible?: boolean;
  areaFill?: boolean;    // area charts
  fillColor?: string;
  fillOpacity?: number;
  smooth?: boolean;
}

// Bar family
interface BarChartDataPoint {
  id?: string | number;
  category: string;
  value: number;
  color?: string;
  data?: any;
}
interface BarChartSeries { id: string; name?: string; color?: string; data: BarChartDataPoint[]; }
interface StackedBarSeries { id?: string | number; name?: string; data: BarChartDataPoint[]; color?: string; visible?: boolean; }

// Pie / Donut
interface PieChartDataPoint {
  id?: string | number;
  value: number;
  label: string;
  color?: string;
  data?: any;
  style?: PieChartSliceStyle;   // stroke, fillOpacity, cornerRadius, gradient, shadow
}
interface DonutChartDataPoint { id?: string | number; label: string; value: number; color?: string; data?: any; }
interface DonutChartRing {
  id?: string | number;
  label?: string;
  data: DonutChartDataPoint[];
  padAngle?: number; startAngle?: number; endAngle?: number;
  thickness?: number; thicknessRatio?: number; innerRadiusRatio?: number;
  colorPalette?: string[]; showInLegend?: boolean;
}

// Scatter
interface ScatterSeries {
  id?: string; name?: string; color?: string;
  data: ChartDataPoint[];
  pointSize?: number; pointColor?: string;
}

// Sparkline
interface SparklinePoint { x: number; y: number; }
// SparklineChartProps.data: number[] | SparklinePoint[]
// Key props: color, fill, fillOpacity, strokeWidth, smooth, showPoints,
//   domain?: { x?: [number, number]; y?: [number, number] },
//   highlightLast, highlightExtrema (bool or SparklineExtremaHighlight),
//   valueFormatter, liveTooltip, multiTooltip,
//   thresholds?: SparklineThreshold[], bands?: SparklineBand[],
//   animation?: { enabled?, duration?, delay?, easing?: 'linear' | 'easeOutCubic' | 'easeInOutCubic' | 'easeOutQuad' }
```

## Notable per-chart props

- `LineChart`: `lineColor`, `lineThickness`, `lineStyle`, `smooth`, `fill`,
  `fillColor`, `fillOpacity`, `areaFillMode: 'single' | 'series'`,
  `enableSeriesToggle`, `xScaleType`/`yScaleType: 'linear' | 'log' | 'time'`,
  `enableBrushZoom` (shift+drag, web), `annotations: ChartAnnotation[]`,
  `decimationThreshold`, `disableAnimations`.
- `BarChart`: `barColor`, `barSpacing`, `barBorderRadius`,
  `orientation: 'vertical' | 'horizontal'`, `layout: 'single' | 'grouped' | 'stacked'`,
  `stackMode: 'normal' | '100%'`, `valueFormatter`, `valueLabel` (config),
  `thresholds: BarChartThreshold[]`, `colorScale(ctx)`, `legendToggleEnabled`.
- `AreaChart`: everything from `LineChart` plus `layout`, `areaOpacity`,
  `stackOrder: 'normal' | 'reverse'`.
- `PieChart`: `innerRadius`, `outerRadius`, `startAngle`, `endAngle`, `padAngle`,
  `showLabels`, `labelPosition`/`labelStrategy`, `wrapLabels`, `showLeaderLines`,
  `labelFormatter`, `showValues`, `valueFormatter(value, total)`,
  `highlightOnHover`, `onSliceHover`, `layers`, `legendToggleEnabled`,
  `keyboardNavigation`, `ariaLabelFormatter`.
- `DonutChart`: `size`, `innerRadiusRatio`, `thickness`, `ringGap`,
  `centerLabel`/`centerSubLabel` (string or formatter), `centerValueFormatter`,
  `renderCenterContent(ctx)`, `isolateOnClick`, `labels: DonutChartLabelsConfig`,
  `inheritColorByLabel`, `primaryRingIndex`, `legendRingIndex`, `emptyLabel`.
- `ScatterChart`: `pointSize`, `pointColor`, `pointOpacity`, `allowAddPoints`,
  `showTrendline: boolean | 'overall' | 'per-series'`, `trendlineColor`,
  `quadrants: ScatterQuadrantConfig` (`x`, `y`, `fills`, `labels`, `showLines`).

## Shared config objects (BaseChartProps and friends)

Every chart extends `BaseChartProps`:
`width`, `height`, `testID`, `style`, spacing (`m mt mr mb ml mx my p pt pr pb pl px py`),
`accessible`, `accessibilityLabel`, `accessibilityHint`, `accessibilityRole`,
`importantForAccessibility`, `animationDuration`, `animationEasing`, `disabled`,
`title`, `subtitle`, `useOwnInteractionProvider`, `suppressPopover`.

Cartesian charts also take:

```ts
xAxis / yAxis: ChartAxis   // show, color, thickness, showTicks, ticks[], tickColor,
                           // tickLength, showLabels, labelFormatter, labelColor,
                           // labelFontSize, title, titleColor, titleFontSize
grid: ChartGrid            // show, color, thickness, style, showMajor/Minor, major/minorLines
legend: ChartLegend        // show, position: 'top'|'bottom'|'left'|'right',
                           // align: 'start'|'center'|'end', items?, textColor, fontSize
tooltip: ChartTooltip<T>   // show, formatter(dataPoint) => string | ReactNode,
                           // backgroundColor, textColor, fontSize, borderRadius, padding
animation: ChartAnimation  // duration, delay, easing, stagger,
                           // type: 'fade'|'scale'|'slide'|'draw'|'drawOn'|'spiral'|'bounce'|'elastic'|'wave'
annotations: ChartAnnotation[]  // shape: 'vertical-line'|'horizontal-line'|'point'|'range'|'text'|'box'
```

Interaction callbacks (`ChartInteractionCallbacks<T>`): `onPress(event)`,
`onDataPointPress(dataPoint, event)` where `event: ChartInteractionEvent<T>`
carries `nativeEvent`, normalized `chartX`/`chartY`, `dataX`/`dataY`,
`dataPoint`, `distance`.

## Theme API

```ts
import { ChartThemeProvider, useChartTheme } from '@platform-blocks/charts';
import type { ChartTheme, HostThemeBridge } from '@platform-blocks/charts';

interface HostThemeBridge {
  textPrimary?: string;
  textSecondary?: string;
  background?: string;
  grid?: string;
  accentPalette?: string[];
  fontFamily?: string;
}

interface ChartTheme {
  colors: { textPrimary; textSecondary; background; grid; accentPalette: string[] };
  fontSize: { xs: number; sm: number; md: number; lg: number };
  radius: number;
  fontFamily?: string;
}
```

`<ChartThemeProvider value={partialTheme} hostThemeBridge={bridge}>` merges
defaults ← `value` ← `hostThemeBridge` (bridge wins). Built-in categorical
palettes (8 hues, fixed slot order, CVD-validated) exist for light and dark
surfaces; the provider picks the dark set automatically when the resolved
`background` is dark and no explicit `accentPalette` is given. The active
palette is also pushed into the module color scheme via
`setDefaultColorScheme(palette)` (exported, along with `colorSchemes` and
`getColorFromScheme`).

## Interaction engine

```ts
import {
  ChartsProvider, GlobalChartsRoot,        // shared interaction root (alias)
  ChartActiveTooltip,                       // the single shared popover
  ChartGestureSurface, useChartPointer,     // lower-level pointer plumbing
  useOptionalChartInteraction, useElementOffset, normalizePointer,
} from '@platform-blocks/charts';
```

`ChartsProviderProps`: `config?: InteractionConfig`, `withPopover?: boolean`
(default true — auto-renders `ChartActiveTooltip`), plus `ViewProps`.

`InteractionConfig` (all optional): `enablePanZoom` (default true),
`zoomMode: 'x' | 'y' | 'both'` (default 'x'), `minZoom` (0.1),
`wheelZoomStep` (0.1), `resetOnDoubleTap` (true), `enableWheelZoom` (false),
`wheelZoomPixelThreshold`, `wheelMinZoom` (0.05), `clampToInitialDomain`,
`enableCrosshair`, `liveTooltip`, `multiTooltip`, `invertPinchZoom`,
`invertWheelZoom`, `popoverPortal` (web, default true), `pointerRAF` (true),
`pointerPixelThreshold`, `aggregatorMaxSeries` (8).

`ChartActiveTooltipProps`: `render(target, rows)`,
`renderEntry(target, defaultNode)`, `renderHeader(rows)`,
`filterEntry(target, index, all)`, `sortEntries(a, b)`, `maxEntries`, `style`,
`offset` (default `{ x: 14, y: 14 }`). Rows are `ActiveTarget`s:
`{ seriesId, markId, kind, datum, pixel, value, distance, label?, color?, formattedValue?, dataX?, dataY?, ... }`.

Per-chart opt-in to a shared context: `useOwnInteractionProvider={false}` and
`suppressPopover` (auto-suppressed when the former is false and this is
undefined).

## Hooks

| Hook | Signature / returns |
|---|---|
| `useStreamingData(initialData?, { maxDataPoints?, updateInterval?, smoothTransitions?, onDataOverflow? })` | `{ data, addDataPoint(point \| points[]), startStreaming(), stopStreaming(), clearData(), isStreaming, queueSize }` |
| `useChartData(data?, series?)` | `{ normalizedSeries, xDomain, yDomain, flattenedData, isEmpty }` (memoized normalization + domains) |
| `useDataDecimation(data, threshold = 1000)` | LTOB-decimated `ChartDataPoint[]` preserving visual trend |
| `useDomains({ xDomain, yDomain })` | current `{ x, y }` domains from the interaction context (initializes them if unset) |
| `usePanZoom(state, setState, opts)` | `{ startPan, updatePan, endPan, startPinch, updatePinch, endPinch, wheelZoom }` — low-level domain math for custom charts |
| `useChartAnimation(data, { duration?, delay?, stagger?, disabled?, easing? })` | Reanimated shared values with staggered progress |

## Building blocks (for custom charts)

`ChartRoot`, `ChartContainer`, `ChartTitle`, `ChartLegend`, `ChartGrid`,
`Axis`, `ChartPlot`, `ChartLayer`, hit-testing (`createHitTester`,
`PointSeriesHitTester`, `BandCategoryHitTester`, `CellGridHitTester`,
`AngularSliceHitTester`, `RadarAxisHitTester`), scale/geometry utilities
(`scaleLinear`, `scaleLog`, `scaleTime`, `generateTicks`, `generateLogTicks`,
`generateTimeTicks`, `createSmoothPath`, `findClosestDataPoint`, ...).

Generated per-chart docs (props tables + examples) live at
`https://platform-blocks.com/components/<ChartName>` — e.g.
`/components/LineChart`.
