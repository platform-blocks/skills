# plocks theming API reference

Verified against `@plocks/ui` source (`packages/ui/src/core/theme/`).
Everything in "Root exports" imports from `'@plocks/ui'`; items marked
*internal* exist in the source but are not re-exported from the package root.

## The theme object (`PlocksTheme`)

```ts
interface PlocksTheme {
  /** Brand base color (hex), e.g. '#3B82F6'. */
  primaryColor: string;

  /** 'light' | 'dark' — presence of this key is what marks a theme "complete". */
  colorScheme: 'light' | 'dark';

  /** Optional bundle of raw design tokens (motion/shadow/radius/spacing/typography/
      interactive/opacity/component). DEFAULT_THEME sets it to DESIGN_TOKENS;
      DARK_THEME omits it. */
  designTokens?: typeof DESIGN_TOKENS;

  /** 10-shade ramps, index 0-9. Light themes: [0] lightest -> [9] darkest.
      Dark themes: inverted, so [5] is always the vivid base and higher indices
      always contrast more against the current surface. */
  colors: {
    primary: string[];
    secondary: string[];
    tertiary: string[];
    surface: string[];
    success: string[];
    warning: string[];
    error: string[];
    gray: string[];
    highlight: string[];
    // optional extended ramps:
    pink?: string[]; purple?: string[]; violet?: string[]; cyan?: string[];
    lime?: string[]; sky?: string[]; amber?: string[]; indigo?: string[];
    teal?: string[];
  };

  /** Semantic text colors. */
  text: {
    primary: string;      // high contrast
    secondary: string;    // medium contrast (real copy: clears 4.5:1)
    muted: string;        // low contrast
    disabled: string;
    link: string;
    onPrimary?: string;   // text on primary[5] fills
  };

  /** Semantic background & surface colors. */
  backgrounds: {
    base: string;       // page background
    subtle: string;     // stripes, alternate rows
    surface: string;    // cards, containers
    elevated: string;   // modals, popovers
    border: string;     // hairline separators
  };

  /** Elevation ladder consumed by Surface (and Card/Menu/Popover/Dialog through it).
      Optional — omitted, a ladder is derived from `backgrounds`. */
  surfaces?: SurfaceScale;   // Record<0|1|2|3, { background; border; shadow }>

  /** Interactive state colors (all optional). */
  states?: {
    focusRing?: string;
    textSelection?: string;
    highlightText?: string;        // matched text in autocomplete/search
    highlightBackground?: string;
  };

  fontFamily: string;

  /** All three scales are CSS px STRINGS ('14px'), keys xs..3xl. */
  fontSizes: { xs; sm; md; lg; xl; '2xl'; '3xl': string };
  spacing:   { xs; sm; md; lg; xl; '2xl'; '3xl': string };
  radii:     { xs; sm; md; lg; xl; '2xl'; '3xl': string };

  /** CSS box-shadow strings, keys xs..xl (parsed to RN shadow props on native). */
  shadows: { xs; sm; md; lg; xl: string };

  /** Media-query widths as px strings, keys xs..xl. */
  breakpoints: { xs; sm; md; lg; xl: string };

  motion: {
    easing: { ease; easeIn; easeOut; easeInOut; spring: string };  // cubic-bezier()
    duration: { instant; fast; normal; slow: string };             // '150ms' etc.
  };

  /** Semantic aliases. */
  semantic: {
    accent: string;
    borderDefault: string;
    borderSubtle: string;
    surfaceElevated: string;
    surfaceCard: string;
    focusRing: string;
  };

  /** Per-component override point: { [componentName]: { defaults?, variants?, sizes? } } */
  components: Record<string, ComponentTokenOverride>;

  /** Escape hatch. DEFAULT_THEME/DARK_THEME put zIndices and elevations here. */
  other: Record<string, any>;
}

type PlocksThemeOverride = Partial<PlocksTheme>;
```

Surface types (also root-exported: `SurfaceProps`, `SurfaceLevel`):

```ts
type SurfaceLevel = 0 | 1 | 2 | 3;            // 0 page, 1 card, 2 dropdown, 3 dialog
type SurfaceShadowToken = 'none' | 'xs' | 'sm' | 'md' | 'lg' | 'xl';
interface SurfaceToken { background: string; border: string; shadow: SurfaceShadowToken }
type SurfaceScale = Record<SurfaceLevel, SurfaceToken>;
```

Built-in values: `DEFAULT_THEME` (light — white cards on `#F7F8FA` page, elevation by
shadow) and `DARK_THEME` (dark — elevation by lighter fills `#0E0E11 -> #2F2F34` plus
hairline borders). Both are root exports. The provider's dark theme is composed
internally as `BUILT_IN_DARK_THEME = { ...DEFAULT_THEME, ...DARK_THEME, colorScheme: 'dark' }`
(*internal* constant; `DEFAULT_THEME` backfills keys `DARK_THEME` omits, e.g.
`designTokens`).

## PlocksProvider

```tsx
<PlocksProvider
  theme={...}                       // PlocksThemeOverride | PlocksThemePair
  colorSchemeMode="auto"            // 'auto' | 'light' | 'dark' (default 'auto')
  themeModeConfig={...}             // ThemeModeConfig — enables useThemeMode()
  withCSSVariables={true}           // web: inject --plocks-* variables
  cssVariablesSelector=":root"      // where variables are declared
  withGlobalCSS={true}              // web: global CSS for lightHidden/darkHidden props
  inherit={true}                    // merge partial theme over parent provider theme
  // non-theming props: withOverlays, withSafeAreaProvider,
  // locale, fallbackLocale, i18nResources, direction, haptics
>
```

Theme resolution (exact source behavior):

1. A `theme` pair selects its `light` or `dark` override for the current scheme.
   A single override without `colorScheme` follows the active scheme; an override
   with `colorScheme` pins it. Keep the object referentially stable.
2. Effective scheme = `themeModeConfig` present ? `actualColorScheme`
   from the internally mounted `ThemeModeProvider` : `colorSchemeMode` prop;
   `'auto'` resolves via the OS `useColorScheme()`. `'dark'` → built-in dark theme,
   `'light'` → `DEFAULT_THEME`.
3. Web side effect (always): sets `data-plocks-color-scheme="light|dark"`
   on `document.documentElement`.

Nest `PlocksProvider` to scope a subtree theme. It accepts
`{ theme?, inherit = true, children }`:

- no `theme` → parent theme (or `DEFAULT_THEME`) passed through unchanged;
- `theme` has truthy `colorScheme` → used directly as a complete theme (no merge);
- otherwise → `mergeTheme(base, theme)` deep-merge, where base = parent theme when
  `inherit` (default), else `DEFAULT_THEME`.

## Theme creation helpers

```ts
createTheme(override: PlocksThemeOverride): PlocksThemeOverride
// Identity function that exists for type checking. Root export.

mergeTheme(defaultTheme, override?): PlocksTheme
// Deep merge (objects merged recursively, arrays REPLACED wholesale — a partial
// color ramp does not splice into the default ramp). *Internal*, not a root
// export: the provider applies it for you; in app code use object spreads.
```

## Color-scheme mode (ThemeModeProvider / useThemeMode)

```ts
type ColorSchemeMode = 'light' | 'dark' | 'auto';

interface ThemeModeConfig {
  initialMode?: ColorSchemeMode;               // default 'auto'
  persistence?: {                              // default: localStorage (web only;
    get: () => ColorSchemeMode | null;         //   key 'plocks-theme-mode';
    set: (mode: ColorSchemeMode) => void;      //   no-op on native)
  };                                           // get() MUST be synchronous
  domConfig?: {                                // web only; defaults:
    selector: string;                          //   'html'
    lightClass: string;                        //   'plocks-light'
    darkClass: string;                         //   'plocks-dark'
    attribute: string;                         //   'data-plocks-manual'
  };
}

// Public hook; PlocksProvider mounts the mode provider when configured:
function useThemeMode(): {
  mode: ColorSchemeMode;                  // the user's choice, may be 'auto'
  setMode(mode: ColorSchemeMode): void;   // also persists via persistence.set
  cycleMode(): void;                      // light -> dark -> auto -> light
  actualColorScheme: 'light' | 'dark';    // resolved ('auto' -> OS scheme)
}
```

Behavior notes (from source):

- `useThemeMode()` **throws** outside a `ThemeModeProvider`. Passing
  `themeModeConfig` to `PlocksProvider` mounts one for you.
- Persisted mode is read synchronously in the state initializer but only applied
  after hydration (SSR/static web renders `initialMode` first, then re-renders
  before paint — pairs with a pre-hydration script for flash-free dark, see the
  plocks-setup skill).
- DOM effect per `domConfig`: `mode === 'auto'` removes the attribute and both
  classes; a manual mode sets `data-plocks-manual="<mode>"` and adds the
  light/dark class matching `actualColorScheme`.
- A second hook named `useColorScheme` exists inside `ThemeModeProvider.tsx`
  returning `actualColorScheme`, but it is **not** the root export of that name
  (see below).

## Reading the theme

```ts
useTheme(): PlocksTheme
// Full theme. Never throws — returns DEFAULT_THEME when no provider is mounted.

useThemeVisuals(): ThemeVisuals
// { colorScheme, primaryColor, colors, text, backgrounds, states }
// Re-renders only when visual/color slices change.

useThemeLayout(): ThemeLayout
// { fontFamily, fontSizes, spacing, radii, shadows, breakpoints, designTokens }
// Re-renders only when layout tokens change.

useColorScheme(): 'light' | 'dark'
// Root export. The OS/system scheme (Appearance API on native,
// prefers-color-scheme on web), NOT the user's in-app choice.
// For the resolved in-app scheme use useThemeMode().actualColorScheme.
```

## Surfaces / elevation

`Surface` component (root export) — the "paper" primitive; Card, Menu, Popover,
Dialog build on it:

```tsx
<Surface
  level={1}          // 0-3; drives background+border+shadow as a set
  raised             // instead of level: parent's level + 1 (clamped at 3)
  withBorder="auto"  // boolean | 'auto' (default): border only in dark mode
  borderColor="..."  borderWidth={1}   // overrides (imply a border)
  bg="..."           // override fill: CSS color, backgrounds key, or palette name/shade
  padding="md"       // size token or px; none by default
  radius="lg"        // BorderRadiusProps; default 'md'
  shadow="sm"        // ShadowProps override
/>

useSurfaceLevel(): SurfaceLevel   // level of the nearest enclosing Surface
```

Resolution helpers in `surfaces.ts` (*internal*, not root-exported —
`resolveSurface(theme, level)`, `resolveSurfaceBackground`, `surfaceInteractionTint(theme,
'band' | 'hover' | 'pressed' | 'selected')`, `clampSurfaceLevel`, `SURFACE_LEVELS`):
prefer the theme's `surfaces` scale, fall back per-field to a ladder derived from
`backgrounds` (base → surface → elevated), so partial custom themes still resolve.
Interaction tints are translucent overlays: white alpha in dark mode, black alpha in
light, so hover/pressed reads correctly at every elevation.

## Variant roles

```ts
type VariantRole = 'filled' | 'outline' | 'light' | 'subtle' | 'surface' | 'gradient';
const CORE_COLORS = ['primary', 'secondary', 'success', 'warning', 'error', 'gray'];

resolveVariantRoles(theme, {
  variant = 'filled',
  color = 'primary',      // CORE_COLORS token -> ramp; anything else -> raw color
  gradientStops?,         // [string, string] for the 'gradient' variant
}): { fill: string; border: string; text: string }
```

- `filled`: solid `ramp[5]` fill, text via `readableTextOn`.
- `outline`: transparent fill, strong border, ramp-contrast text.
- `light` / `subtle`: alpha tint of the strong color over the surface
  (dark surfaces get slightly heavier tints); text picked by measured contrast
  against the *composited* background.
- `surface`: neutral recessed well from background tokens (ignores `color`).
- `gradient`: caller supplies stops; picks a legible text color.

`resolveGradientStops(theme, color)` (*internal*): canonical tight two-stop gradient
`[ramp[5], ramp[7]]` (custom colors: same hue darkened).

## Color utilities (root exports)

```ts
withAlpha(hex, alpha): string        // '#3B82F6', 0.14 -> 'rgba(59, 130, 246, 0.14)'
readableTextOn(fill, onLight = '#1A1A1A', onDark = '#FFFFFF'): string
                                     // prefers light text on saturated fills (>= 3:1)
contrastRatio(a, b): number          // WCAG ratio 1..21
composite(fg, bg, alpha): string     // opaque hex result of an alpha tint over bg
pickReadable(candidates, surface, min = 4.5): string
                                     // first candidate clearing `min`, else best
```

*Internal* siblings in `colorUtils.ts`: `normalizeHex`, `adjustHexColor`,
`hexToRgb`, `relativeLuminance`. All utilities pass non-hex input through unchanged
(luminance falls back to 0.5).

## Numeric size scales (for RN styles)

Theme tokens are px *strings*; RN style objects want numbers. Root exports from
`sizes.ts`: `resolveSize(value, scale, unit?)`, `getSpacing`, `getIconSize`,
`getHeight`, `getLineHeight`, `SIZE_SCALES`, `COMPONENT_SIZES`. Scales (all keyed
xs..3xl): `fontSize` 10-24, `spacing` 4-32, `iconSize` 12-40, `height` 28-68,
`radius` 2-20, plus `lineHeight`, `controlLabel`, `controlIcon`. The radius/shadow
component helpers in `radius.ts`/`shadow.ts` (`getBorderRadius`, `RADIUS_SCALE` with
`none`/`full`/`chip`, `createShadowStyles`, `COMPONENT_SHADOW_DEFAULTS`) are
*internal* — components accept `radius`/`shadow` props instead.

## Web CSS variables

When `withCSSVariables` (default true), a `<style data-plocks-variables>`
tag is injected declaring, under `cssVariablesSelector` (default `:root`):

```
--plocks-color-scheme            light | dark
--plocks-primary-color
--plocks-color-{ramp}-{0..9}     e.g. --plocks-color-primary-5
--plocks-font-family
--plocks-bg-base | -bg-subtle | -bg-surface | -bg-elevated
--plocks-border-color
--plocks-focus-ring | -text-selection | -highlight-text
--plocks-highlight-background    (states vars only when defined)
--plocks-font-size-{xs..3xl}
--plocks-spacing-{xs..3xl}
--plocks-radius-{xs..3xl}
--plocks-shadow-{xs..xl}
--plocks-breakpoint-{xs..xl}
```

Side effects: `::selection` background from the text-selection var; `document.body`
gets the theme's base background and a matching text color inline. The variables
re-inject whenever the theme changes. Selectors available on `<html>`:
`[data-plocks-color-scheme="dark"]` (always, resolved scheme) and — only
under `themeModeConfig` with a manual mode — `.plocks-dark` /
`.plocks-light` and `[data-plocks-manual]`.

## Root export map (theming)

From `'@plocks/ui'`: `PlocksProvider`, `BUILT_IN_DARK_THEME`, `useTheme`,
`useThemeVisuals`, `useThemeLayout`, `useThemeMode`,
`useColorScheme`, `createTheme`, `DEFAULT_THEME`, `DARK_THEME`, `resolveVariantRoles`,
`CORE_COLORS`, `withAlpha`, `readableTextOn`, `contrastRatio`, `composite`,
`pickReadable`, `Surface`, `useSurfaceLevel`, `resolveSurface`,
`surfaceInteractionTint`, `resolveSpacing`, `resolveRadius`,
`resolveFontSize`; types
`PlocksTheme`, `PlocksThemeOverride`, `PlocksProviderProps`,
`PlocksThemePair`, `ThemeVisuals`, `ThemeLayout`, `ThemeModeConfig`,
`ColorSchemeMode`, `ColorScheme`, `VariantRole`, `VariantRoles`,
`ResolveVariantOptions`, `SurfaceProps`, `SurfaceLevel`, `SizeValue`, `SizeToken`.

Not root-exported (do not tell users to import): `mergeTheme`,
`CSSVariables`, `resolveSurfaceBackground`,
`clampSurfaceLevel`, `SURFACE_LEVELS`, `resolveGradientStops`, `adjustHexColor`,
`relativeLuminance`, `normalizeHex`, `hexToRgb`.
