---
name: platform-blocks-layout
description: Build screen layouts with the @platform-blocks/ui React Native library. Use when composing screens with Flex/Row/Column/Grid, building responsive breakpoint-based layouts (GridItem span={{base, sm, md, lg, xl}}, useBreakpoint), applying spacing system props (p, px, py, m, mt, gap), assembling app shell / navigation chrome (AppShell, defineAppLayout blueprints), or laying out content in Cards, Surfaces, and Blocks.
---

# Platform Blocks Layout

Layout primitives from `@platform-blocks/ui` for React Native (iOS/Android/Web).
Everything imports from the package root:

```tsx
import { Flex, Row, Column, Grid, GridItem, Card, Surface, Block, AppShell } from '@platform-blocks/ui';
```

`references/api.md` has full prop tables; `references/patterns.md` has complete copy-paste screens.

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
      <Text colorVariant="secondary">Subtitle</Text>
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

`BreakpointProvider` is mounted automatically by `PlatformBlocksProvider` — no setup needed.
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
