# @plocks/charts — Copy-paste patterns

All examples assume the peers are installed
(`react-native-svg`, `react-native-reanimated`) and run on iOS/Android/Web.

## 1. App wiring: theme bridge from @plocks/ui

The pattern plocks.dev itself uses — a small bridge component between
`PlocksProvider` and the app so every chart tracks the UI theme
(including dark mode) automatically:

```tsx
import React from 'react';
import { PlocksProvider, useTheme } from '@plocks/ui';
import { ChartThemeProvider } from '@plocks/charts';

// Categorical series palette — fixed hue order, assigned by slot, never cycled.
// Pinned hex values validated (lightness band, chroma floor, CVD separation,
// 3:1 contrast) against both the light and dark chart surfaces, so a series
// keeps the same color across a theme toggle.
const CHART_SERIES_PALETTE = [
  '#3B82F6', // blue
  '#16A34A', // green
  '#A855F7', // purple
  '#D97706', // amber
  '#0891B2', // cyan
  '#65A30D', // lime
  '#6366F1', // indigo
  '#EF4444', // red
];

const ChartThemeBridge: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const theme = useTheme();

  const hostBridge = React.useMemo(() => ({
    textPrimary: theme.text.primary,
    textSecondary: theme.text.secondary,
    background: theme.backgrounds.surface,
    grid: theme.colors.gray?.[3] ?? theme.backgrounds.border ?? '#e5e7eb',
    accentPalette: CHART_SERIES_PALETTE,
    fontFamily: theme.fontFamily,
  }), [theme]);

  return (
    <ChartThemeProvider hostThemeBridge={hostBridge}>
      {children}
    </ChartThemeProvider>
  );
};

export const AppProviders: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <PlocksProvider>
    <ChartThemeBridge>{children}</ChartThemeBridge>
  </PlocksProvider>
);
```

Without `@plocks/ui`, pass the bridge values from any design system —
all `HostThemeBridge` fields are optional, and omitting `accentPalette` lets
the provider auto-select its built-in light- or dark-surface palette based on
`background`:

```tsx
<ChartThemeProvider
  hostThemeBridge={{
    textPrimary: '#e6e6e6',
    textSecondary: '#9a9a9a',
    background: '#1a1a19', // dark surface → built-in dark palette auto-selected
    grid: '#333330',
    fontFamily: 'Inter',
  }}
>
  <App />
</ChartThemeProvider>
```

## 2. Multi-series line chart (axes, legend, tooltip, crosshair)

```tsx
import { LineChart } from '@plocks/charts';

<LineChart
  title="Weekly signups"
  h={280}
  series={[
    { name: 'iOS',     data: [{ x: 1, y: 40 }, { x: 2, y: 55 }, { x: 3, y: 48 }, { x: 4, y: 70 }] },
    { name: 'Android', data: [{ x: 1, y: 30 }, { x: 2, y: 42 }, { x: 3, y: 61 }, { x: 4, y: 58 }] },
  ]}
  smooth
  showPoints
  enableCrosshair
  liveTooltip
  multiTooltip
  enableSeriesToggle
  xAxis={{ title: 'Week', labelFormatter: (v) => `W${v}` }}
  yAxis={{ title: 'Signups' }}
  grid={{ show: true, style: 'dashed' }}
  legend={{ show: true, position: 'bottom' }}
  tooltip={{ formatter: (p) => `${p.y} signups` }}
  annotations={[
    { id: 'target', shape: 'horizontal-line', y: 50, label: 'Target', color: '#D97706' },
  ]}
  onDataPointPress={(point) => console.log('pressed', point)}
  accessibilityLabel="Line chart of weekly signups for iOS and Android"
/>
```

## 3. Bar chart: single, stacked, value labels, threshold

```tsx
import { BarChart } from '@plocks/charts';

// Single series with labels + reference line
<BarChart
  h={260}
  data={[
    { category: 'Mon', value: 12 },
    { category: 'Tue', value: 19 },
    { category: 'Wed', value: 8 },
    { category: 'Thu', value: 24 },
  ]}
  barBorderRadius={4}
  valueLabel={{ show: true, position: 'outside', formatter: (v) => `${v}` }}
  thresholds={[{ value: 15, label: 'Goal', style: 'dashed', color: '#EF4444' }]}
  yAxis={{ title: 'Deploys' }}
/>

// Stacked (or 'grouped'), normalized to 100%
<BarChart
  h={260}
  data={[]}
  series={[
    { id: 'web',    name: 'Web',    data: [{ category: 'Q1', value: 30 }, { category: 'Q2', value: 45 }] },
    { id: 'mobile', name: 'Mobile', data: [{ category: 'Q1', value: 50 }, { category: 'Q2', value: 40 }] },
  ]}
  layout="stacked"
  stackMode="100%"
  legend={{ show: true, position: 'top' }}
  legendToggleEnabled
  multiTooltip
/>
```

For horizontal bars add `orientation="horizontal"`. Dedicated
`GroupedBarChart` / `StackedBarChart` components take
`series: StackedBarSeries[]` directly.

## 4. Donut with center content

```tsx
import { DonutChart } from '@plocks/charts';

<DonutChart
  size={260}
  data={[
    { label: 'Passed',  value: 182 },
    { label: 'Flaky',   value: 14 },
    { label: 'Failed',  value: 6 },
  ]}
  innerRadiusRatio={0.7}
  padAngle={2}
  centerLabel="Tests"
  centerValueFormatter={(value, total) => `${Math.round((value / total) * 100)}%`}
  legend={{ show: true, position: 'right' }}
  isolateOnClick
  labels={{ show: true, position: 'outside', showPercentage: true, minAngle: 12 }}
  accessibilityLabel="Donut chart of test results: 182 passed, 14 flaky, 6 failed"
/>
```

`PieChart` covers the same ground with `innerRadius`/`outerRadius`, plus
`keyboardNavigation` and `ariaLabelFormatter` for accessibility-first slices.

## 5. Sparkline KPI row

```tsx
import { View } from 'react-native';
import { SparklineChart } from '@plocks/charts';

const revenue = [12, 14, 11, 18, 16, 22, 25];
const errors  = [3, 2, 4, 2, 6, 5, 3];

<View style={{ flexDirection: 'row', gap: 16 }}>
  <SparklineChart
    w={120}
    h={36}
    data={revenue}
    smooth
    fill
    highlightLast
    valueFormatter={(v) => `$${v}k`}
    domain={{ y: [0, 30] }}          // shared domain avoids re-scaling jitter
    animation={{ duration: 600, easing: 'easeOutCubic' }}
  />
  <SparklineChart
    w={120}
    h={36}
    data={errors}
    color="#EF4444"
    highlightExtrema={{ showMax: true }}
    thresholds={[{ value: 5, dashed: true, label: 'SLO' }]}
    bands={[{ from: 0, to: 3, opacity: 0.08 }]}
    domain={{ y: [0, 8] }}
  />
</View>
```

## 6. Scatter with quadrants and trendline

```tsx
import { ScatterChart } from '@plocks/charts';

<ScatterChart
  h={300}
  series={[
    { name: 'Cohort A', data: [{ x: 2, y: 8 }, { x: 5, y: 3 }, { x: 7, y: 9 }] },
    { name: 'Cohort B', data: [{ x: 3, y: 4 }, { x: 6, y: 7 }, { x: 8, y: 2 }] },
  ]}
  showTrendline="per-series"
  quadrants={{
    x: 5,
    y: 5,
    showLines: true,
    labels: { topRight: 'Invest', bottomRight: 'Maintain', topLeft: 'Improve', bottomLeft: 'Divest' },
    fillOpacity: 0.05,
  }}
  liveTooltip
  enableCrosshair
/>
```

## 7. Pan/zoom over a time series

```tsx
import { LineChart } from '@plocks/charts';

const DAY = 24 * 60 * 60 * 1000;
const start = Date.parse('2026-01-01');
const data = Array.from({ length: 365 }, (_, i) => ({
  x: start + i * DAY,                       // numeric timestamps for time scale
  y: 100 + Math.sin(i / 12) * 20 + i * 0.1,
}));

<LineChart
  h={300}
  data={data}
  xScaleType="time"
  xAxis={{ labelFormatter: (v) => new Date(v).toLocaleDateString(undefined, { month: 'short' }) }}
  enablePanZoom
  zoomMode="x"
  enableWheelZoom
  resetOnDoubleTap
  clampToInitialDomain
  enableBrushZoom               // shift+drag on web
  decimationThreshold={500}     // LTOB decimation above 500 points
  onDomainChange={(xDomain, yDomain) => console.log('visible', xDomain)}
/>
```

## 8. Streaming / live data

```tsx
import React, { useEffect } from 'react';
import { LineChart, useStreamingData } from '@plocks/charts';

function LiveChart() {
  const { data, addDataPoint, startStreaming, stopStreaming } = useStreamingData([], {
    maxDataPoints: 120,        // rolling window
    updateInterval: 250,       // batch flush cadence (ms)
    onDataOverflow: (removed) => console.log('dropped', removed.length),
  });

  useEffect(() => {
    startStreaming();
    const id = setInterval(() => {
      addDataPoint({ x: Date.now(), y: 50 + Math.random() * 30 });
    }, 100);                   // points arrive faster than the flush; the hook batches
    return () => { clearInterval(id); stopStreaming(); };
  }, [addDataPoint, startStreaming, stopStreaming]);

  return (
    <LineChart
      h={240}
      data={data}
      xScaleType="time"
      xAxis={{ labelFormatter: (v) => new Date(v).toLocaleTimeString() }}
      disableAnimations         // skip entry animation for continuously moving data
    />
  );
}
```

## 9. Dashboard: multiple charts, one shared tooltip + zoom

```tsx
import {
  ChartsProvider, ChartActiveTooltip, LineChart, ScatterChart,
} from '@plocks/charts';
import { Text } from 'react-native';

<ChartsProvider
  config={{ multiTooltip: true, enableCrosshair: true, enableWheelZoom: true, zoomMode: 'x' }}
  withPopover={false}                       // we mount a customized popover below
>
  <LineChart
    useOwnInteractionProvider={false}       // join the shared context
    suppressPopover
    h={220}
    series={[{ name: 'Latency p50', data: latencyP50 }]}
  />
  <ScatterChart
    useOwnInteractionProvider={false}
    suppressPopover
    h={220}
    data={incidents}
  />
  <ChartActiveTooltip
    maxEntries={5}
    offset={{ x: 12, y: 12 }}
    renderHeader={(rows) => (
      <Text style={{ fontWeight: '600' }}>
        {rows[0]?.dataX != null ? new Date(rows[0].dataX).toLocaleTimeString() : ''}
      </Text>
    )}
    filterEntry={(target) => target.value != null}
  />
</ChartsProvider>
```

Leave `withPopover` at its default (`true`) and drop the manual
`ChartActiveTooltip` when the default popover is fine.

## 10. Accessibility baseline

```tsx
<PieChart
  data={slices}
  accessible
  accessibilityLabel="Browser market share: Chrome 62%, Safari 20%, Firefox 8%, other 10%"
  accessibilityHint="Swipe or use arrow keys to move between slices"
  keyboardNavigation
  ariaLabelFormatter={(slice, pct) => `${slice.label}: ${pct.toFixed(1)} percent`}
/>
```

Every chart accepts `accessible`, `accessibilityLabel`, `accessibilityHint`,
`accessibilityRole`, and `importantForAccessibility` (from `BaseChartProps`).
Summarize the actual data in the label — the SVG itself is not readable by
screen readers.
