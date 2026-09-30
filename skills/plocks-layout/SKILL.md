---
name: plocks-layout
description: Build screen layouts with the @plocks/ui React Native library. Use when composing screens with Flex/Row/Column/Grid, building responsive breakpoint-based layouts (GridItem span={{base, sm, md, lg, xl}}, useBreakpoint), applying spacing system props (p, px, py, m, mt, gap), assembling app shell / navigation chrome (AppShell, defineAppLayout blueprints), or laying out content in Cards, Surfaces, and Blocks.
---

# plocks Layout

Layout primitives from `@plocks/ui` for React Native (iOS/Android/Web).
Everything imports from the package root:

```tsx
import { Flex, Row, Column, Grid, GridItem, Card, Surface, Block, AppShell } from '@plocks/ui';
```

Four reference files sit alongside this one:

- `references/api.md` — the curated API: how the pieces compose, the union
  types, the defaults that bite. **Read this first.**
- `references/props.md` — generated, exhaustive prop tables for every component
  in this skill. Look here for the complete surface of a single prop.
- `references/icons.md` — generated, every `name` the built-in `Icon` registry
  accepts. Check it before writing `<Icon name="…">`; unlisted names render
  nothing.
- `references/patterns.md` — complete copy-paste screens.

## Core primitives

| Component | What it is | Key defaults |
| --- | --- | --- |
| `Flex` | Flexbox `View` | `direction="row"`, `align="flex-start"`, `justify="flex-start"`, `wrap="nowrap"`, `gap="sm"` |
| `Row` | `Flex` with `direction="row"` | `gap="sm"` |
| `Column` | `Flex` with `direction="column"` | `gap="sm"`, **`fullWidth` defaults to `true`** |
| `Grid` / `GridItem` | 12-column responsive grid | `columns={12}`, `gap={0}` (no gap by default) |
| `Block` | Polymorphic styled box (`bg`, `radius`, `shadow`, positioning) | `gap="sm"` when flex |
| `Card` | Padded surface with variants + `Card.Section` | `variant="filled"`, `padding="md"` (12px), `radius="md"` |
| `Surface` | Elevation-ladder container (`level` 0–3) | no padding, `withBorder="auto"` |
| `AppShell` | App chrome: header/navbar/aside/footer/bottomNav/main | responsive, safe-area aware |

Typical screen skeleton:

```tsx
<ScrollView contentContainerStyle={{ padding: 20 }}>
  <Column gap="xl">
    <Column gap="sm">
      <Title order={1}>Screen title</Title>
      <Text c="secondary">Subtitle</Text>
    </Column>
    <Card variant="elevated" p="lg">
      <Column gap="md">{/* card content */}</Column>
    </Card>
  </Column>
</ScrollView>
```

## Spacing system props

All layout components (Flex/Row/Column/Grid/GridItem/Card/Surface/Block/Masonry) accept
`SpacingProps`: `m mx my mt mr mb ml` and `p px py pt pr pb pl`.
Values (`SpacingValue`): a token `'xs'|'sm'|'md'|'lg'|'xl'|'2xl'|'3xl'`, a number
(raw px), `'auto'`, or `0`. Token scale: xs=4, sm=8, md=12, lg=16, xl=20, 2xl=24, 3xl=32.

`gap` / `rowGap` / `columnGap` take a `SizeValue` — the same tokens or a number
(`gap="md"` or `gap={16}`). Percentages/strings like `'10%'` are NOT valid gap/spacing values.

Sizing props (`LayoutProps`, on Flex/Row/Column/Card/Surface): `fullWidth`, `w`, `h`,
`maxW`, `minW`, `maxH`, `minH` (React Native `DimensionValue` — numbers or `'50%'`).
`w` wins over `fullWidth`.

RTL is handled for you: `ml/mr/pl/pr` map to logical start/end, and `Flex` row
directions mirror automatically (opt out with `disableRTLMirroring`).

## Responsive layouts

**Grid span/columns** accept a plain number or a mobile-first breakpoint object with keys
`{ base, sm, md, lg, xl }` (no `xs` key here). Thresholds for Grid resolution
(`DEFAULT_BREAKPOINTS`): base 0, sm 480, md 640, lg 960, xl 1200.

```tsx
<Grid columns={12} gap="md">
  <GridItem span={{ base: 12, md: 6, lg: 4 }}>
    <Card variant="elevated" p="md">…</Card>
  </GridItem>
</Grid>
```

**Breakpoint hook** for conditional rendering — `useBreakpoint()` returns
`'xs' | 'sm' | 'md' | 'lg' | 'xl'` (thresholds: xs 0, sm 576, md 768, lg 992, xl 1200):

```tsx
const breakpoint = useBreakpoint();
const isMobile = breakpoint === 'xs' || breakpoint === 'sm';
return isMobile ? <Column gap="md">{panes}</Column> : <Row gap="lg">{panes}</Row>;
```

`BreakpointProvider` is mounted automatically by `PlocksProvider` — no setup needed.
Helpers `resolveResponsiveProp(value, width)` and `resolveResponsiveValue(value, breakpoint)`
are exported for resolving responsive objects manually.

**AppShell responsive sizes** (`headerHeight`, `navbarWidth`, …) use `ResponsiveSize`
objects that additionally allow an `xs` key: `{ base?, xs?, sm?, md?, lg?, xl? }`.

## Cards and surfaces

- `Card` variants: `'filled'` (default) `'outline'` `'elevated'` `'subtle'` `'ghost'` `'gradient'`.
  Inner padding via `padding` (token/number, default `'md'`) — the spacing prop `p` also
  works and overrides it. `onPress` makes the whole card pressable. `withBorder` adds a
  1px border on any variant. `Card.Section` (direct child only) escapes the card padding
  for full-bleed images/banded rows; set `clip` on the Card to round its corners.
- `Surface` drives background + border + shadow together from an elevation `level` (0–3);
  `raised` puts a nested surface one level above its parent automatically.
- `Block` is the low-level styled box: `bg`, `radius` (incl. `'full'`), `shadow`,
  `opacity`, absolute `position`/`top`/`left`/`zIndex`, `fluid` (flex:1), polymorphic
  `component` prop.

## App shell / navigation chrome

Two ways to build app chrome:

1. **Component composition** — `<AppShell header={{height:60}} navbar={{width:240, breakpoint:'md'}}>`
   with children `AppShell.Header`, `AppShell.Navbar`, `AppShell.Aside`, `AppShell.Footer`,
   `AppShell.BottomNav`, `AppShell.Main`, `AppShell.Section` — or `autoLayout` with
   `headerContent`/`navbarContent`/`bottomNavItems` props. Handles safe areas, mobile
   drawer vs desktop rail, collapse/expand-on-hover. Hooks: `useAppShell()` (layout
   context: `isMobile`, `breakpoint`, `toggleNavbar`…), `useNavbarHover()`.

2. **Layout blueprints** — the declarative engine. `defineAppLayout({...})` describes the
   whole shell (sections as `{ component | render, props, show(ctx) }` entries, responsive
   `breakpoints`, `main` config, `overlays`); mount it once with
   `<AppLayoutProvider blueprint={...} value={{ query, pathname, navigation }}>` wrapping
   `<AppLayoutRenderer>{routes}</AppLayoutRenderer>`. Every entry receives an
   `AppLayoutRuntimeContext` (`breakpoint`, `isMobile`, `orientation`, `platform`,
   `pathname`, `query`, `theme`, `colorScheme`) so visibility and props can react to
   route and viewport. See patterns.md for a full blueprint.

## Pitfalls (verified against source)

1. **No `flex` prop on Flex/Row/Column** — use `grow={1}` or `style={{ flex: 1 }}`.
   (`Block` does have `fluid` for flex:1.)
2. **`Column` is `fullWidth` by default**; plain `Flex`/`Row` are not — children of a
   `Row` shrink-wrap unless you add `fullWidth` / `grow`.
3. **Flex defaults `align="flex-start"`**, not `stretch` like raw RN Views — set
   `align="stretch"` explicitly when children should fill the cross axis.
4. **`Grid` defaults `gap={0}`** (Flex defaults `gap="sm"`) — pass `gap="md"` explicitly.
5. **`GridItem` without `span` renders at span 1** (1/12 width) — always set `span`.
6. **Grid responsive keys are `{base, sm, md, lg, xl}`** — `xs` is silently ignored in
   Grid `span`/`columns` objects (only AppShell `ResponsiveSize` accepts `xs`).
7. **Two breakpoint scales exist**: Grid resolves at 480/640/960/1200; `useBreakpoint()`
   reports at 576/768/992/1200. Don't expect them to flip at exactly the same width.
8. **`hiddenFrom`/`visibleFrom` exist in the prop types but are not applied by the layout
   primitives** — hide responsively by conditional rendering with `useBreakpoint()`.
9. **`Card.Section` must be a direct child** of `Card` (fragment/View wrappers break the
   first/last padding detection).
10. `gap`/spacing tokens top out at `'3xl'` (32) — for bigger separations use numbers.
11. **`Grid` breaks under static web rendering (SSR / prerender).** `Grid` resolves
    `columns` and `GridItem` `span` against `useWindowDimensions()` at render time.
    During Expo Router's static web export there is no window, so width is `0`, the
    `base` value wins, and those percentage widths are **baked into the exported
    HTML** — and hydration does not patch them, so a prerendered page stays stacked
    at every viewport. For any route that is statically rendered, build breakpoint
    layouts with a wrapping `Flex` + `flexBasis` instead, which needs no width
    measurement:

    ```tsx
    // SSR-correct responsive grid — pure CSS wrapping, no width measurement.
    // flexBasis sets the target column width; flexGrow lets the last row fill.
    <Flex direction="row" wrap="wrap" gap="md">
      {features.map((feature) => (
        <Card
          key={feature.title}
          variant="elevated"
          p="lg"
          style={{ flexBasis: 320, flexGrow: 1 }}
        >
          {/* … */}
        </Card>
      ))}
    </Flex>
    ```

    This is what `universal-template`'s `FeatureGrid` ships, for exactly this
    reason.

    `Grid` is fine in native apps and in client-only web routes. Note that
    `useBreakpoint()` does **not** have this problem — it lives in the other
    breakpoint system (`core/responsive`), which explicitly falls back to a desktop
    width when `window` is absent and recomputes on mount. `Grid` uses
    `core/theme/breakpoints` + `useWindowDimensions()`, which has no such guard.

## Anything this skill does not cover

This skill covers layout primitives, the responsive system, and app chrome.
plocks is much larger — 100+ core UI components, 24 chart types, and 48 documented hooks. Do not
guess an API for something outside this scope; fetch the generated docs instead:

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

- **Navigation components** — `Tabs`, `Breadcrumbs`, `Pagination`, `Stepper`,
  `Menu`, `Spotlight`, `TableOfContents`, `Tree`, `Link`.
- **Data display** — `Table`, `DataTable`, `DataList`, `Timeline`, `Badge`,
  `Chip`, `Avatar`, `Accordion`, `Collapse`, `ListGroup`, `Carousel`, `Markdown`.
- **Overlays and feedback** — `Dialog`, `Popover`, `Tooltip`, `ContextMenu`,
  `Toast`, `Alert`, `Progress`, `Loader`, `Skeleton`, `LoadingOverlay`.
- **Media** — `Image`, `Gallery`, `Video`, `Waveform`, and `AudioPlayer`
  (`import { AudioPlayer } from '@plocks/media'`).
- **Install and provider wiring** → the `plocks-setup` skill. **Theme
  tokens** → `plocks-theming`. **Forms** → `plocks-forms`.
