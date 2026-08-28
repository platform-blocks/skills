---
name: platform-blocks-theming
description: Theme the @platform-blocks/ui React Native library. Use when customizing the Platform Blocks theme (brand colors, ramps, spacing, radii, shadows), implementing dark mode or light/dark/auto color-scheme switching, defining custom color palettes, working with surfaces/elevation, resolving variant colors (filled/outline/light/subtle/surface/gradient), reading theme tokens in custom components, or styling web output via the injected --platform-blocks-* CSS variables and html classes.
---

# Platform Blocks Theming

Platform Blocks theming is a single `PlatformBlocksTheme` object distributed by
`PlatformBlocksProvider` through React context. Two complete themes ship with the
library (`DEFAULT_THEME` light, `DARK_THEME` dark) and the provider switches between
them automatically — unless you hand it a `theme` of your own, which turns that
switching off (see "Theme resolution rules").

All public APIs import from the package root:

```tsx
import {
  PlatformBlocksProvider, PlatformBlocksThemeProvider, ThemeModeProvider,
  useTheme, useThemeVisuals, useThemeLayout, useThemeMode, useColorScheme,
  createTheme, DEFAULT_THEME, DARK_THEME,
  resolveVariantRoles, CORE_COLORS,
  withAlpha, readableTextOn, contrastRatio, composite, pickReadable,
  Surface, useSurfaceLevel,
  type PlatformBlocksTheme, type PlatformBlocksThemeOverride,
  type ThemeModeConfig, type ColorSchemeMode,
} from '@platform-blocks/ui';
```

## Core workflow

1. **Mount the provider once** at the app root. With no `theme` prop it renders
   `DEFAULT_THEME` in light and `{...DEFAULT_THEME, ...DARK_THEME, colorScheme: 'dark'}`
   in dark, following the OS by default (`colorSchemeMode="auto"`).
2. **Add user-controlled light/dark/auto** by passing `themeModeConfig` (see
   `ThemeModeConfig` in api.md). That mounts `ThemeModeProvider` internally and makes
   `useThemeMode()` available anywhere below: `{ mode, setMode, cycleMode, actualColorScheme }`.
3. **Customize colors/tokens** either with a partial override
   (`theme={createTheme({...})}` — merged over `DEFAULT_THEME`, light-only) or with two
   complete brand themes you switch yourself (the only way to keep dark mode with a
   custom theme — full recipe in patterns.md).
4. **Read tokens in custom components** with `useTheme()` (full theme), or the
   granular `useThemeVisuals()` / `useThemeLayout()` slices to avoid unrelated re-renders.
5. **Express elevation** with the `Surface` component (`level={0..3}` or `raised`)
   instead of hand-picking backgrounds; it consumes `theme.surfaces`.
6. **Color custom components** with `resolveVariantRoles(theme, { variant, color })`
   so your fills/borders/text match Button/Chip/Badge on any theme.

## Theme resolution rules (verify against these before advising)

The provider resolves the active theme in this exact order
(`PlatformBlocksProvider.tsx`):

- `theme` prop present → **used as-is for every color scheme**. Built-in light/dark
  switching is bypassed entirely. Downstream, `PlatformBlocksThemeProvider` then:
  - object has a truthy `colorScheme` → treated as a **complete** theme, used directly;
  - otherwise → treated as a partial override and deep-merged over `DEFAULT_THEME`
    (so a partial override is *always light* + your changes, even in dark mode).
- no `theme` prop → scheme comes from `themeModeConfig` (its resolved
  `actualColorScheme`) if provided, else from the `colorSchemeMode` prop
  (`'auto' | 'light' | 'dark'`, default `'auto'` = OS preference); dark resolves to
  the built-in dark theme, light to `DEFAULT_THEME`.
- When `themeModeConfig` is passed, the `colorSchemeMode` prop is ignored.

## Key concepts

**Theme anatomy** — `primaryColor`, `colorScheme`, `colors` (10-shade ramps:
`primary`, `secondary`, `tertiary`, `surface`, `success`, `warning`, `error`, `gray`,
`highlight`, plus optional `pink/purple/violet/cyan/lime/sky/amber/indigo/teal`),
`text` (primary/secondary/muted/disabled/link/onPrimary), `backgrounds`
(base/subtle/surface/elevated/border), optional `surfaces` (elevation ladder 0-3),
optional `states` (focusRing/textSelection/highlightText/highlightBackground),
`fontFamily`, `fontSizes`/`spacing`/`radii` (xs..3xl), `shadows` (xs..xl),
`breakpoints`, `motion`, `semantic` (accent/borderDefault/borderSubtle/
surfaceElevated/surfaceCard/focusRing), `components`, `other`, `designTokens`.
Full annotated shape in references/api.md.

**Ramps invert between schemes.** Light ramps run lightest `[0]` → darkest `[9]`;
dark ramps run the reverse. Index `[5]` is always the vivid base, and a *higher*
index always means *more contrast against the current surface* in both schemes.
Never hardcode "low index = light".

**Surfaces ladder** — `theme.surfaces[0..3]` maps elevation to
`{ background, border, shadow }` as a set: 0 = page, 1 = resting cards, 2 = dropdowns/
popovers, 3 = dialogs/toasts. Light mode conveys elevation with shadow (fill stays
white); dark mode with a lighter fill + hairline border. Themes that omit `surfaces`
get a ladder derived from `backgrounds`. Use `<Surface level={n}>` or
`<Surface raised>` (parent level + 1, clamped at 3, read via `useSurfaceLevel()`).

**Variant roles** — `resolveVariantRoles(theme, { variant, color })` returns
`{ fill, border, text }` for `'filled' | 'outline' | 'light' | 'subtle' | 'surface' |
'gradient'`. `color` is one of `CORE_COLORS`
(`primary/secondary/success/warning/error/gray`) or any raw hex/CSS color. Text is
picked by measured WCAG contrast (`pickReadable`, `readableTextOn`), tints are alpha
washes over the live surface (`withAlpha`), so results stay legible on custom themes.

**Web only** — the provider injects `--platform-blocks-*` CSS variables
(`withCSSVariables`, default true, under `cssVariablesSelector`, default `':root'`)
and always stamps `data-platform-blocks-color-scheme="light|dark"` on `<html>`. With
`themeModeConfig`, a *manual* (non-auto) choice also adds class
`platform-blocks-light` / `platform-blocks-dark` and
`data-platform-blocks-manual="light|dark"` to `<html>` (configurable via
`themeModeConfig.domConfig`). Mode persists to localStorage key
`platform-blocks-theme-mode` by default. Variable list in api.md.

## Pitfalls

1. **Passing any `theme` prop kills built-in dark mode.** To keep switching with
   brand colors, build complete light + dark themes and select between them with
   your own `ThemeModeProvider` wrapper (patterns.md "Custom palette with dark mode").
2. **A partial override never goes dark.** `createTheme({ primaryColor: ... })` has no
   `colorScheme`, so it merges over `DEFAULT_THEME` — permanently light.
3. **`useThemeMode()` throws** unless `themeModeConfig` was passed to
   `PlatformBlocksProvider` (or you mounted `ThemeModeProvider` yourself).
4. **`persistence.get` must be synchronous** (`() => ColorSchemeMode | null`). The
   default localStorage persistence is a no-op on native — use a sync store like
   `react-native-mmkv`; `AsyncStorage` does not fit this API.
5. **Not root exports:** `mergeTheme`, `BUILT_IN_DARK_THEME`, `resolveSurface`,
   `surfaceInteractionTint`, `resolveGradientStops`, `CSSVariables` are internal.
   Compose custom themes with object spreads over `DEFAULT_THEME`/`DARK_THEME`, and
   use the `Surface` component instead of `resolveSurface`.
6. **`DARK_THEME` omits `designTokens`.** When building a custom dark theme, spread
   `...DEFAULT_THEME` first, then `...DARK_THEME` (that is exactly how the built-in
   dark theme is composed).
7. **`theme.spacing`/`radii`/`fontSizes` are CSS px strings** (`'16px'`), not numbers.
   In React Native styles use the numeric helpers (`getSpacing`, `getIconSize`,
   `getHeight`, `resolveSize`, `SIZE_SCALES`) or `parseInt` the token.
8. **Keep a custom `theme` object referentially stable** (module scope or `useMemo`) —
   the provider assumes it is stable and re-renders the whole tree when it changes.
9. **`useTheme()` never throws** — outside a provider it silently returns
   `DEFAULT_THEME`, which can mask a missing provider during debugging.
10. **Don't hand-pick palette indices for text on tinted fills** — contrast flips
    between schemes. Use `resolveVariantRoles`, `pickReadable`, or `readableTextOn`.

## References

- **references/api.md** — full `PlatformBlocksTheme` shape, provider props and
  resolution logic, `ThemeModeConfig`, all hooks, surfaces/variant/color utilities,
  the complete CSS-variable list, and the root export map.
- **references/patterns.md** — complete copy-paste examples: provider setup, theme
  switcher UI, persisted mode config (web + native), custom brand palettes with and
  without dark mode, reading tokens, Surface elevation, variant-aware custom
  components, nested sub-tree themes, and web CSS integration.

## Anything this skill does not cover

This skill covers the theme object, color schemes, palettes, surfaces, and
variant color resolution. Platform Blocks is much larger — 97 components, 25
charts, and 18 hooks. Do not guess an API for something outside this scope;
fetch the generated docs instead:

| What you need | Where |
| --- | --- |
| Index of every page, one line each | `https://platform-blocks.com/llms.txt` |
| One component or chart | `https://platform-blocks.com/llms/components/<Name>.md` |
| One hook | `https://platform-blocks.com/llms/hooks/<useName>.md` |
| Guides | `https://platform-blocks.com/llms/guides/{getting-started,accessibility,localization}.md` |
| Everything in one file (~1.3 MB) | `https://platform-blocks.com/llms-full.txt` |

`<Name>` is the exact PascalCase export name — `.../llms/components/DataTable.md`,
`.../llms/components/AreaChart.md`. Each page carries the component's full prop
table (type, required, default, description) plus runnable examples, generated
from the source, so it is authoritative where memory is not. When you are unsure
whether something exists or what it is called, read `llms.txt` first — it lists
every page with a one-line summary.

Import paths: components come from the package root (`import { X } from
'@platform-blocks/ui'`). The exceptions are subpath-only: `FormLayout`
(`@platform-blocks/ui/FormLayout`), `AudioPlayer`
(`@platform-blocks/ui/AudioPlayer`), and the whole `Navigation` module —
`NavigationContainer`, `createStackNavigator`, `createDrawerNavigator`,
`Screen`, `useNavigation`, `useRoute` (`@platform-blocks/ui/Navigation`). A few
utilities also live on subpaths (e.g. `validationRules` on
`@platform-blocks/ui/Input`). A docs page existing does not guarantee a root
export — `HoverCard`, for instance, is internal and has no page and no export.

Notably outside this skill:

- **Per-component styling props.** Which `variant`/`size`/`colorVariant` values a
  given component accepts is on that component's page — theming defines what the
  values mean, not which ones exist.
- **Install and provider wiring** → the `platform-blocks-setup` skill. **Layout**
  → `platform-blocks-layout`. **Chart theming** → `platform-blocks-charts`.
