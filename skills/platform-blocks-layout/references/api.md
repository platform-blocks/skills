# Platform Blocks layout API reference

All symbols import from `@platform-blocks/ui`. Verified against
`packages/ui/src` (Flex, Layout, Grid, Card, Surface, Block, AppShell,
core/utils, core/theme, core/responsive).

## Shared value types

```ts
// Size tokens used by gap, padding, radius, etc.
type SizeValue = 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl' | '3xl' | number;

// Spacing token scale (px): xs=4, sm=8, md=12, lg=16, xl=20, 2xl=24, 3xl=32
// Radius token scale  (px): xs=2, sm=4,  md=6,  lg=8,  xl=12, 2xl=16, 3xl=20

// Spacing prop values — tokens, raw px numbers, 'auto', or 0
type SpacingValue = SizeValue | 'auto' | '0' | number;

// Responsive object used by Grid columns / GridItem span (NO xs key)
type ResponsiveProp<T> = T | { base?: T; sm?: T; md?: T; lg?: T; xl?: T };
// Resolved against window width using DEFAULT_BREAKPOINTS (mobile-first,
// largest defined key whose min-width <= width wins):
const DEFAULT_BREAKPOINTS = { base: 0, sm: 480, md: 640, lg: 960, xl: 1200 };

// AppShell responsive sizing (numbers, strings like '100%', or object — HAS xs)
type ResponsiveSize = number | string | {
  base?: number | string; xs?: number | string; sm?: number | string;
  md?: number | string; lg?: number | string; xl?: number | string;
};
type Breakpoint = 'base' | 'xs' | 'sm' | 'md' | 'lg' | 'xl';
```

## SpacingProps (on every layout component)

| Prop | Applies | Prop | Applies |
| --- | --- | --- | --- |
| `m` | margin all sides | `p` | padding all sides |
| `mx` | margin left+right | `px` | padding left+right |
| `my` | margin top+bottom | `py` | padding top+bottom |
| `mt` `mr` `mb` `ml` | individual margins | `pt` `pr` `pb` `pl` | individual paddings |

All accept `SpacingValue`. Individual props override shorthands
(`mt` beats `m`/`my`). On web, `ml/mr/pl/pr` emit logical
`marginInlineStart/End`; on native they swap under RTL — so left/right
props behave as start/end.

`SpacingProps` also declares `lightHidden`, `darkHidden`, `hiddenFrom`,
`visibleFrom` — these are extracted but NOT applied as styles by
Flex/Grid/Card etc. Prefer conditional rendering with `useBreakpoint()`.

## LayoutProps (Flex, Row, Column, Card, Surface)

| Prop | Type | Effect |
| --- | --- | --- |
| `fullWidth` | `boolean` | `width: '100%'` |
| `w` / `h` | `DimensionValue` | width / height (`w` overrides `fullWidth`) |
| `maxW` / `minW` | `DimensionValue` | max/min width |
| `maxH` / `minH` | `DimensionValue` | max/min height |

## Flex

`FlexProps extends SpacingProps, LayoutProps, Omit<ViewProps, 'style'>`

| Prop | Type | Default |
| --- | --- | --- |
| `direction` | `'row' \| 'column' \| 'row-reverse' \| 'column-reverse'` | `'row'` |
| `align` | `'flex-start' \| 'flex-end' \| 'center' \| 'stretch' \| 'baseline'` | `'flex-start'` |
| `justify` | `'flex-start' \| 'flex-end' \| 'center' \| 'space-between' \| 'space-around' \| 'space-evenly'` | `'flex-start'` |
| `wrap` | `'nowrap' \| 'wrap' \| 'wrap-reverse'` | `'nowrap'` |
| `gap` | `SizeValue` | `'sm'` |
| `rowGap` / `columnGap` | `SizeValue` | — |
| `grow` | `number` | — (there is NO `flex` prop) |
| `shrink` | `number` | — |
| `basis` | `DimensionValue` | — |
| `disableRTLMirroring` | `boolean` | `false` (row directions auto-mirror in RTL) |
| `children` / `style` / `testID` | usual | — |

Other `ViewProps` (`onLayout`, accessibility props, `pointerEvents`, …) forward
to the underlying `View`.

## Row and Column

```ts
interface RowProps    extends Omit<FlexProps, 'direction'> { direction?: 'row' | 'row-reverse' }
interface ColumnProps extends Omit<FlexProps, 'direction'> { direction?: 'column' | 'column-reverse' }
```

- `Row`: `Flex` with `direction="row"`, `gap="sm"`.
- `Column`: `Flex` with `direction="column"`, `gap="sm"`, **`fullWidth={true}` by default**
  (pass `fullWidth={false}` to shrink-wrap).

## Grid and GridItem

`GridProps extends SpacingProps` (no LayoutProps; has its own `fullWidth`):

| Prop | Type | Default |
| --- | --- | --- |
| `columns` | `ResponsiveProp<number>` | `12` |
| `gap` | `SizeValue` | `0` |
| `rowGap` / `columnGap` | `SizeValue` | falls back to `gap` |
| `fullWidth` | `boolean` | `false` |
| `children` / `style` / `testID` | usual | — |

`GridItemProps extends SpacingProps`:

| Prop | Type | Default |
| --- | --- | --- |
| `span` | `ResponsiveProp<number>` | resolves to `1` — always set it |

Behavior: Grid is a wrapping row; each valid child is wrapped in a View whose
width is `span / columns * 100%`. `gap` becomes horizontal padding (with a
negative margin on the container) and `rowGap` becomes `marginBottom` on every
item (so the grid has trailing bottom space of one rowGap). Responsive `span` /
`columns` resolve against `useWindowDimensions().width` with
`DEFAULT_BREAKPOINTS` (480/640/960/1200) — re-resolves live on resize/rotation.

## Responsive utilities (root exports)

| Export | Signature | Notes |
| --- | --- | --- |
| `useBreakpoint()` | `() => Breakpoint` | Returns `'xs'\|'sm'\|'md'\|'lg'\|'xl'` (never `'base'` in practice). Thresholds: xs 0, sm 576, md 768, lg 992, xl 1200. Listens to resize on web; reads `Dimensions` once on native. |
| `BreakpointProvider` | component | Shared resize listener; mounted automatically by `PlatformBlocksProvider`. |
| `resolveResponsiveProp(value, width, breakpoints?)` | `ResponsiveProp<T> → T` | Width-based resolver Grid uses (`DEFAULT_BREAKPOINTS`). |
| `resolveResponsiveValue(value, breakpoint)` | `ResponsiveSize → number` | Breakpoint-token resolver AppShell uses. |
| `DEFAULT_BREAKPOINTS` | object | `{ base:0, sm:480, md:640, lg:960, xl:1200 }` |

Note the two scales differ (Grid: 480/640/960/1200 vs useBreakpoint:
576/768/992/1200). `useIsMobile`/`useResponsiveValue` exist internally but are
NOT exported from the package root.

## Card

`CardProps extends SpacingProps, LayoutProps, BorderRadiusProps { radius }, ShadowProps { shadow }`

| Prop | Type | Default |
| --- | --- | --- |
| `variant` | `'filled' \| 'outline' \| 'elevated' \| 'subtle' \| 'ghost' \| 'gradient'` | `'filled'` |
| `padding` | `SizeValue` | `'md'` (12px). Spacing prop `p` overrides it. |
| `radius` | `SizeValue` (+ radius tokens) | `'md'` |
| `shadow` | shadow token \| `'none'` | per-variant default |
| `withBorder` | `boolean` | adds 1px theme border on any variant |
| `borderColor` / `borderWidth` | `string` / `number` | imply `withBorder` |
| `bg` | color string or palette name (`'primary'`, `'success'`, …) | variant background |
| `clip` | `boolean` | `false` — clip children to radius (needed for full-bleed sections) |
| `onPress` / `disabled` | pressable card | renders `Pressable` when `onPress` set |
| `accessibilityRole` / `accessibilityLabel` / `accessibilityState` | RN a11y | role defaults to button when pressable |

`Card.Section` (also exported as `CardSection`) — must be a DIRECT child:

| Prop | Type | Notes |
| --- | --- | --- |
| `withBorder` | `boolean` | divider borders (top unless first, bottom unless last) |
| `inheritPadding` | `boolean` | horizontal padding equal to card's |
| `py` / `px` | `SizeValue` | section's own padding (`px` overrides `inheritPadding`) |

## Surface

`SurfaceProps extends SpacingProps, LayoutProps, BorderRadiusProps, ShadowProps, Omit<ViewProps,'style'>`

| Prop | Type | Default |
| --- | --- | --- |
| `level` | `0 \| 1 \| 2 \| 3` | inherits from enclosing Surface |
| `raised` | `boolean` | parent level + 1 (clamped at 3) |
| `withBorder` | `boolean \| 'auto'` | `'auto'` — hairline border in dark mode only |
| `borderColor` / `borderWidth` | overrides | imply a border |
| `bg` | color / `theme.backgrounds` key / palette | wins over level fill |
| `padding` | `SizeValue` | none by default |

## Block

Polymorphic styled box: `BlockProps extends SpacingProps, BlockStyleProps`,
plus `component` (element type to render as, default div/View) and `className` (web).

Key `BlockStyleProps`: `bg`, `radius` (`SizeValue` tokens or `'full'`),
`borderWidth`, `borderColor`, `shadow` (0–5 or tokens), `opacity`,
`w`/`h` (number/string/`'auto'`/`'full'`), `fullWidth`, `fluid` (**flex: 1**),
`minW/minH/maxW/maxH`, `grow`/`shrink` (boolean or number), `basis`,
`direction`, `align`, `justify`, `wrap` (boolean or value),
`gap` (default `'sm'`, pass `0` to remove), `position` (`'relative' | 'absolute'`),
`top/right/bottom/left/start/end`, `zIndex`, `flex` (boolean — render as flex container).

## Masonry

`MasonryProps extends SpacingProps`: `data: MasonryItem[]` (`{ id, content, heightRatio?, style? }`),
`numColumns` (default 2), `gap: SizeValue`, `renderItem`, `loading`, `emptyContent`,
FlashList passthrough props. Use for staggered card feeds.

## AppShell

`AppShellProps extends SpacingProps` — main configs (all sizes are `ResponsiveSize`):

| Prop | Type / shape |
| --- | --- |
| `layout` | `'default' \| 'alt'` |
| `header` | `{ height, collapsed?, offset?, zIndex? }` |
| `navbar` | `{ width, breakpoint, collapsed?: {mobile?, desktop?}, collapsedWidth? (default 72), expandOnHover?, expandOnHoverPush?, autoExpandBreakpoint?, startCollapsedDesktop?, zIndex? }` |
| `aside` | `{ width, breakpoint, collapsed?, zIndex? }` |
| `footer` | `{ height, collapsed?, offset?, zIndex? }` |
| `bottomNav` | `{ height, showOnlyMobile?, collapsed?, zIndex? }` |
| `layoutSections` | `{ header?, navbar?, aside?, footer?, bottomNav? : boolean }` |
| `autoLayout` | `boolean` — AppShell composes sections from the `*Content` props |
| `headerContent` / `navbarContent` / `asideContent` / `footerContent` | `ReactNode \| () => ReactNode` |
| `bottomNavItems` | `BottomAppBarItem[]` (`{ key, label, icon, activeIcon?, badgeCount?, onPress? }`) |
| `mobileMenu` | `{ type?: 'modal'\|'drawer'\|'fullscreen', animationType?, showBackdrop?, closeOnOutsidePress?, transitionDuration? }` |
| `statusBar` | `{ style?, backgroundColor?, translucent?, hidden? }` |
| `maxContentWidth` / `centerContent` | constrain + center main content |
| `tableOfContents` / `hideTableOfContentsOnMobile` / `tableOfContentsWidth` / `tableOfContentsWithBorder` | right-hand TOC rail |
| `padding`, `withBorder`, `withSafeArea` (default true), `backgroundColor`, `transitionDuration`, `disabled` | shell chrome |

Sub-components (compound + named exports): `AppShell.Header`, `AppShell.Navbar`
(`drawerMode?`), `AppShell.Aside`, `AppShell.Footer`, `AppShell.BottomNav`
(`items`, `activeKey`, `onItemPress`, `variant: 'solid'|'surface'|'elevated'|'translucent'`,
`fab`, `showLabels`), `AppShell.Main` (`maxWidth`, `centerContent`, `tableOfContents`,
`tocWidth`…), `AppShell.Section` (`grow`, `withScrollArea`), `AppShell.MobileMenu`,
`AppShell.StatusBarManager`, `BottomAppBar`, `StatusBarManager`.

Hooks: `useAppShell()` → `{ headerHeight, navbarWidth, asideWidth, footerHeight,
bottomNavHeight, isNavbarCollapsed, isMobile, breakpoint, openNavbar, closeNavbar,
toggleNavbar, navbarOpen, … }`; also `useAppShellApi()`, `useAppShellLayout()`,
`useNavbarHover()`.

## App layout blueprints (declarative shell)

Exports: `defineAppLayout`, `AppLayoutProvider`, `AppLayoutRenderer`,
`useAppLayoutContext`, plus the types below.

```ts
interface AppLayoutBlueprint {
  id: string;
  breakpoints?: { headerHeight?, navbarWidth?, asideWidth?, footerHeight?,
                  bottomNavHeight?, padding?: ResponsiveSize };
  header?:    LayoutEntry;                       // { component, props?, show?, key?, target? }
  navbar?:    LayoutNavbarConfig;                // entry + width/collapsedWidth/expandOnHover/
                                                 //   expandOnHoverPush/autoExpandBreakpoint/startCollapsedDesktop
  aside?:     LayoutAsideConfig;                 // entry + width
  footer?:    LayoutFooterConfig;                // entry + height
  bottomNav?: LayoutBottomNavConfig;             // entry + height
  overlays?:  LayoutEntry[];                     // target: 'shell' | 'root' | 'root-before' | 'root-after'
  effects?:   ((ctx) => void | (() => void))[];
  main?:      { id?, role?, maxWidth?, centerContent?, tableOfContents?, props? };
  layout?:    { withSafeArea?, withBorder?, backgroundColor?, padding?, statusBar?,
                transitionDuration?, disabled?, testID? };
  visibility?: Partial<Record<'header'|'navbar'|'aside'|'footer'|'bottomNav', (ctx) => boolean>>;
  meta?: Record<string, unknown>;
}

// Entries are either { component, props }, or { render: (ctx) => ReactNode };
// `props` may be a function of ctx; `show(ctx)` toggles the section.

interface AppLayoutRuntimeContext {
  blueprint; query; pathname?; navigation?;   // navigation: { push?, replace?, goBack?, open? }
  platform: PlatformOSType;
  breakpoint: Breakpoint; isMobile: boolean;
  isLandscape: boolean; orientation: 'portrait' | 'landscape';
  theme: any; colorScheme?: string; reducedMotion: boolean;
  meta?: Record<string, unknown>;
}
```

`AppLayoutProvider` takes `blueprint` and `value: { query?, pathname?, navigation?,
platform?, meta? }` (`AppLayoutRuntimeOverrides`); `AppLayoutRenderer` renders the
shell and puts its children in `AppShell.Main`.
