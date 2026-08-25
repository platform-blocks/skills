# Platform Blocks theming patterns

Complete, copy-paste examples. All imports are from the package root
`'@platform-blocks/ui'`.

## 1. Built-in light/dark following the OS

The zero-config default. No `theme` prop means the provider switches between the
built-in light and dark themes; `colorSchemeMode` defaults to `'auto'` (OS).

```tsx
import { PlatformBlocksProvider } from '@platform-blocks/ui';

export default function App() {
  return (
    <PlatformBlocksProvider>
      {/* your app */}
    </PlatformBlocksProvider>
  );
}
```

Force a scheme instead: `<PlatformBlocksProvider colorSchemeMode="dark">`.

## 2. User-controlled light/dark/auto with persistence

Pass `themeModeConfig` to enable `useThemeMode()`. This real-world config (from the
official expo-template `app/_layout.tsx`) persists to localStorage on web and falls
back to plain auto on native:

```tsx
import { useMemo } from 'react';
import { Platform } from 'react-native';
import { PlatformBlocksProvider, type ThemeModeConfig } from '@platform-blocks/ui';

const THEME_STORAGE_KEY = 'platform-blocks-theme-mode';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  const themeModeConfig: ThemeModeConfig = useMemo(() => {
    if (Platform.OS === 'web' && typeof window !== 'undefined') {
      return {
        initialMode: 'auto',
        persistence: {
          get: () => {
            try {
              const stored = window.localStorage.getItem(THEME_STORAGE_KEY);
              if (stored === 'light' || stored === 'dark' || stored === 'auto') return stored;
            } catch {
              return null;
            }
            return null;
          },
          set: mode => {
            try {
              window.localStorage.setItem(THEME_STORAGE_KEY, mode);
            } catch {
              /* noop */
            }
          },
        },
      };
    }
    return { initialMode: 'auto' };
  }, []);

  return (
    <PlatformBlocksProvider themeModeConfig={themeModeConfig}>
      {children}
    </PlatformBlocksProvider>
  );
}
```

Notes:

- `persistence.get` must be **synchronous** — `AsyncStorage` cannot back it. For
  native persistence use a sync store, e.g. `react-native-mmkv`:

```ts
import { MMKV } from 'react-native-mmkv';
import type { ColorSchemeMode, ThemeModeConfig } from '@platform-blocks/ui';

const storage = new MMKV();

const themeModeConfig: ThemeModeConfig = {
  initialMode: 'auto',
  persistence: {
    get: () => {
      const stored = storage.getString('theme-mode');
      return stored === 'light' || stored === 'dark' || stored === 'auto'
        ? (stored as ColorSchemeMode)
        : null;
    },
    set: mode => storage.set('theme-mode', mode),
  },
};
```

- For flash-free dark mode on statically rendered web (pre-hydration script in
  `+html.tsx` sharing the same storage key and class names), see the
  `platform-blocks-setup` skill.

## 3. Theme switcher UI (SegmentedControl)

From the expo-template settings screen. Requires `themeModeConfig` (pattern 2).

```tsx
import { Card, Column, SegmentedControl, Text, Title, useThemeMode } from '@platform-blocks/ui';

export default function AppearanceSettings() {
  const { mode, setMode } = useThemeMode();

  return (
    <Card variant="elevated" p="lg">
      <Column gap="md">
        <Title order={3}>Appearance</Title>
        <Text colorVariant="secondary">
          Auto follows the OS setting. Your choice is saved and applied on next launch.
        </Text>
        <SegmentedControl
          value={mode}
          onChange={value => setMode(value as typeof mode)}
          data={[
            { value: 'light', label: 'Light' },
            { value: 'dark', label: 'Dark' },
            { value: 'auto', label: 'Auto' },
          ]}
        />
      </Column>
    </Card>
  );
}
```

A one-button variant: `const { cycleMode } = useThemeMode()` cycles
light → dark → auto. `useThemeMode().actualColorScheme` gives the resolved
`'light' | 'dark'` (never `'auto'`) for conditional rendering.

## 4. Light-only brand override (partial theme)

A partial override (no `colorScheme` key) deep-merges over `DEFAULT_THEME`.
**This permanently disables dark mode** — any `theme` prop bypasses built-in
switching, and a partial merges over the light theme. Use it only for light-only
apps; otherwise use pattern 5. Define the object at module scope (the provider
assumes it is referentially stable).

```tsx
import { PlatformBlocksProvider, createTheme } from '@platform-blocks/ui';

// A ramp is 10 shades, lightest [0] -> darkest [9]; [5] is the vivid base.
const VIOLET = [
  '#F5F3FF', '#EDE9FE', '#DDD6FE', '#C4B5FD', '#A78BFA',
  '#8B5CF6', '#7C3AED', '#6D28D9', '#5B21B6', '#4C1D95',
];

const brandTheme = createTheme({
  primaryColor: VIOLET[5],
  colors: { primary: VIOLET },        // arrays replace wholesale — supply all 10
  text: { link: VIOLET[5] },          // nested objects merge key-by-key
  semantic: { accent: VIOLET[5], focusRing: VIOLET[4] },
  radii: { md: '10px', lg: '14px' },
});

export default function App() {
  return (
    <PlatformBlocksProvider theme={brandTheme}>
      {/* always light + your overrides */}
    </PlatformBlocksProvider>
  );
}
```

## 5. Custom palette WITH dark mode (complete theme pair)

To keep light/dark switching with brand colors, build two **complete** themes
(spread the built-ins) and pick between them yourself. Mount `ThemeModeProvider`
directly so `useThemeMode()` still works everywhere; pass the resolved scheme back
as `colorSchemeMode` so the web `data-platform-blocks-color-scheme` attribute stays
correct. Do not also pass `themeModeConfig` here (that would mount a second,
nested mode provider).

```tsx
import {
  PlatformBlocksProvider, ThemeModeProvider, useThemeMode,
  DEFAULT_THEME, DARK_THEME,
  type PlatformBlocksTheme, type ThemeModeConfig,
} from '@platform-blocks/ui';

const VIOLET = [
  '#F5F3FF', '#EDE9FE', '#DDD6FE', '#C4B5FD', '#A78BFA',
  '#8B5CF6', '#7C3AED', '#6D28D9', '#5B21B6', '#4C1D95',
];
// Dark ramps are inverted: darkest [0] -> lightest [9], base stays at [5].
const VIOLET_DARK = [...VIOLET].reverse();

const brandLight: PlatformBlocksTheme = {
  ...DEFAULT_THEME,
  primaryColor: VIOLET[5],
  colors: { ...DEFAULT_THEME.colors, primary: VIOLET },
  text: { ...DEFAULT_THEME.text, link: VIOLET[5] },
  semantic: { ...DEFAULT_THEME.semantic, accent: VIOLET[5], focusRing: VIOLET[4] },
};

const brandDark: PlatformBlocksTheme = {
  ...DEFAULT_THEME,   // first: backfills keys DARK_THEME omits (designTokens, ...)
  ...DARK_THEME,      // exactly how the built-in dark theme is composed
  colorScheme: 'dark',
  primaryColor: VIOLET_DARK[5],
  colors: { ...DARK_THEME.colors, primary: VIOLET_DARK },
  text: { ...DARK_THEME.text, link: VIOLET_DARK[5] },
  semantic: { ...DARK_THEME.semantic, accent: VIOLET_DARK[5] },
};

function BrandedProvider({ children }: { children: React.ReactNode }) {
  const { actualColorScheme } = useThemeMode();
  return (
    <PlatformBlocksProvider
      theme={actualColorScheme === 'dark' ? brandDark : brandLight}
      colorSchemeMode={actualColorScheme}
    >
      {children}
    </PlatformBlocksProvider>
  );
}

const modeConfig: ThemeModeConfig = { initialMode: 'auto' }; // add persistence per pattern 2

export default function App() {
  return (
    <ThemeModeProvider config={modeConfig}>
      <BrandedProvider>{/* your app */}</BrandedProvider>
    </ThemeModeProvider>
  );
}
```

## 6. Reading theme tokens in a custom component

```tsx
import { View, Text as RNText } from 'react-native';
import { useTheme, getSpacing, SIZE_SCALES } from '@platform-blocks/ui';

export function StatCard({ label, value }: { label: string; value: string }) {
  const theme = useTheme();

  return (
    <View
      style={{
        backgroundColor: theme.backgrounds.surface,
        borderColor: theme.backgrounds.border,
        borderWidth: 1,
        // theme.spacing/radii/fontSizes are px STRINGS ('16px') — use the numeric
        // scales/helpers for RN style values:
        borderRadius: SIZE_SCALES.radius.lg,       // 8
        padding: getSpacing('lg'),                 // 16
      }}
    >
      <RNText style={{ color: theme.text.secondary, fontSize: SIZE_SCALES.fontSize.sm }}>
        {label}
      </RNText>
      <RNText style={{ color: theme.text.primary, fontSize: SIZE_SCALES.fontSize['2xl'] }}>
        {value}
      </RNText>
    </View>
  );
}
```

Performance: subscribe to just the slice you use —
`useThemeVisuals()` (colorScheme, primaryColor, colors, text, backgrounds, states)
or `useThemeLayout()` (fontFamily, fontSizes, spacing, radii, shadows, breakpoints,
designTokens) — so color changes don't re-render layout-only components and vice
versa. Scheme checks: `useTheme().colorScheme === 'dark'`.

## 7. Elevation with Surface

Never hand-pick elevated backgrounds — dark mode expresses elevation with lighter
fills and hairline borders, light mode with shadows. `Surface` handles both:

```tsx
import { Surface, useSurfaceLevel } from '@platform-blocks/ui';

function Panel() {
  return (
    <Surface level={1} padding="lg" radius="lg">
      {/* resting card on the page */}
      <Surface raised padding="md" radius="md">
        {/* parent level + 1 (here: 2) — a popover/well inside the card,
            no hard-coded numbers, clamps at 3 */}
      </Surface>
    </Surface>
  );
}

function DebugLevel() {
  const level = useSurfaceLevel(); // level of the nearest enclosing Surface
  return null;
}
```

Custom themes can define the ladder explicitly (`theme.surfaces`), one
`{ background, border, shadow }` token per level 0-3:

```ts
const themeWithSurfaces: PlatformBlocksTheme = {
  ...DEFAULT_THEME,
  surfaces: {
    0: { background: '#FAF9F7', border: '#F1EFEA', shadow: 'none' }, // page
    1: { background: '#FFFFFF', border: '#E9E6DF', shadow: 'xs' },   // cards
    2: { background: '#FFFFFF', border: '#E9E6DF', shadow: 'md' },   // dropdowns
    3: { background: '#FFFFFF', border: '#E9E6DF', shadow: 'xl' },   // dialogs
  },
};
```

Themes that omit `surfaces` still work: a ladder is derived from
`backgrounds.base/surface/elevated/border`.

## 8. Variant-aware custom component

Resolve fill/border/text the same way Button/Chip/Badge do, so your component obeys
`variant` + `color` on any theme, both schemes, with measured-contrast text:

```tsx
import { Pressable, Text as RNText } from 'react-native';
import {
  useTheme, resolveVariantRoles, withAlpha,
  type VariantRole,
} from '@platform-blocks/ui';

interface TagProps {
  label: string;
  variant?: VariantRole;              // 'filled' | 'outline' | 'light' | 'subtle' | 'surface' | 'gradient'
  color?: string;                     // 'primary' | 'success' | ... | any raw hex
  onPress?: () => void;
}

export function Tag({ label, variant = 'light', color = 'primary', onPress }: TagProps) {
  const theme = useTheme();
  const roles = resolveVariantRoles(theme, { variant, color });

  return (
    <Pressable
      onPress={onPress}
      style={({ pressed }) => ({
        backgroundColor: pressed ? withAlpha(roles.text, 0.12) : roles.fill,
        borderColor: roles.border,
        borderWidth: 1,
        borderRadius: 999,
        paddingHorizontal: 12,
        paddingVertical: 4,
      })}
    >
      <RNText style={{ color: roles.text, fontWeight: '500' }}>{label}</RNText>
    </Pressable>
  );
}

// <Tag label="Beta" />                             — primary tint
// <Tag label="Failed" variant="filled" color="error" />
// <Tag label="Custom" variant="outline" color="#0EA5E9" />  — raw colors work too
```

Related root exports: `contrastRatio(a, b)`, `pickReadable(candidates, surface)`,
`readableTextOn(fill)`, `composite(fg, bg, alpha)` for hand-rolled color logic.

## 9. Nested sub-tree theme

`PlatformBlocksThemeProvider` can re-theme a section. A partial override merges
over the *parent's* current theme (`inherit` defaults to true), so it survives
light/dark switching above it:

```tsx
import { PlatformBlocksThemeProvider, createTheme } from '@platform-blocks/ui';

const dangerZone = createTheme({
  primaryColor: '#EF4444',
  semantic: { accent: '#EF4444' },
});

function DangerSettings() {
  return (
    <PlatformBlocksThemeProvider theme={dangerZone}>
      {/* Buttons etc. in here use red as "primary" */}
    </PlatformBlocksThemeProvider>
  );
}
```

Caveat: if the nested `theme` object has a `colorScheme` key it is treated as a
complete theme and replaces the parent theme entirely (no merge).

## 10. Web: CSS variables and scheme selectors

With the default `withCSSVariables`, plain CSS (and non-RN DOM islands) can consume
the live theme; the variables re-inject on every theme change:

```css
.marketing-hero {
  background: var(--platform-blocks-bg-surface);
  color: var(--platform-blocks-color-primary-5);
  border: 1px solid var(--platform-blocks-border-color);
  border-radius: var(--platform-blocks-radius-lg);
  padding: var(--platform-blocks-spacing-xl);
  box-shadow: var(--platform-blocks-shadow-sm);
  font-family: var(--platform-blocks-font-family);
}

/* Resolved scheme — always present on <html> (covers 'auto'): */
html[data-platform-blocks-color-scheme='dark'] .marketing-hero {
  background: var(--platform-blocks-bg-elevated);
}

/* Manual choice only — classes exist only when themeModeConfig is used AND the
   user picked light/dark explicitly (removed again in 'auto'): */
html.platform-blocks-dark .promo-banner { display: none; }
html[data-platform-blocks-manual='light'] .os-hint { display: none; }
```

Scope the variables to a subtree instead of `:root` with
`<PlatformBlocksProvider cssVariablesSelector="#pb-app">`, or disable injection
entirely with `withCSSVariables={false}`. The class names/attribute are
customizable via `themeModeConfig.domConfig` (`selector`, `lightClass`,
`darkClass`, `attribute`).
