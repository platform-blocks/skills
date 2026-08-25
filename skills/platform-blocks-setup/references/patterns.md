# Platform Blocks setup — copy-paste patterns

All code below is taken verbatim from the official templates
(`platform-blocks/expo-template`, `platform-blocks/expo-min-template`) and works
against `@platform-blocks/ui@1.0.0`.

## Minimal app root (no router) — `App.tsx`

From expo-min-template. `SafeAreaProvider` > `PlatformBlocksProvider` > app:

```tsx
import { StatusBar } from 'expo-status-bar';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { Button, Card, Column, PlatformBlocksProvider, Text, Title } from '@platform-blocks/ui';

export default function App() {
  return (
    <SafeAreaProvider>
      <PlatformBlocksProvider>
        <StatusBar style="auto" />
        <Column style={{ flex: 1 }} justify="center" align="center" p="lg" gap="lg">
          <Card variant="elevated" p="lg" style={{ maxWidth: 480, width: '100%' }}>
            <Column gap="md">
              <Title order={1}>Hello, Platform Blocks 👋</Title>
              <Text colorVariant="secondary">
                Edit App.tsx to start building. The provider is already set up, so every
                component, hook, and theme token is ready to use.
              </Text>
              <Button
                title="Read the docs"
                variant="filled"
                onPress={() => {
                  console.log('https://platform-blocks.com/getting-started');
                }}
              />
            </Column>
          </Card>
        </Column>
      </PlatformBlocksProvider>
    </SafeAreaProvider>
  );
}
```

## Expo Router root layout — `app/_layout.tsx`

From expo-template. Provider + persisted `ThemeModeConfig` + `ContentReveal`
(pairs with `+html.tsx` below) + `NavigationThemeBridge` (feeds Platform Blocks
theme tokens into React Navigation):

```tsx
import { useEffect, useMemo, type ReactNode } from 'react';
import { Platform } from 'react-native';
import {
  DarkTheme,
  DefaultTheme,
  Stack,
  ThemeProvider as NavigationThemeProvider,
} from 'expo-router';
import { StatusBar } from 'expo-status-bar';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import {
  PlatformBlocksProvider,
  useTheme,
  useThemeMode,
  type ThemeModeConfig,
} from '@platform-blocks/ui';

const THEME_STORAGE_KEY = 'platform-blocks-theme-mode';

/**
 * Lifts the `platform-blocks-content-pending` class the pre-hydration script in
 * app/+html.tsx stamps on <html> for dark-theme readers. The class is only ever
 * added when the script resolved the scheme to dark, so this waits until the
 * provider has actually rendered the dark theme before revealing — dark-mode
 * visitors see dark content appear, never a light flash. The script's own
 * timer is the fallback if hydration fails entirely.
 */
function ContentReveal() {
  const { actualColorScheme } = useThemeMode();

  useEffect(() => {
    if (Platform.OS !== 'web' || typeof document === 'undefined') {
      return;
    }
    const root = document.documentElement;
    if (!root.classList.contains('platform-blocks-content-pending')) {
      return;
    }
    if (actualColorScheme !== 'dark') {
      return;
    }
    const reveal = () => {
      root.classList.remove('platform-blocks-content-pending');
    };
    if (typeof requestAnimationFrame === 'function') {
      const frame = requestAnimationFrame(reveal);
      return () => {
        cancelAnimationFrame(frame);
        reveal();
      };
    }
    reveal();
  }, [actualColorScheme]);

  return null;
}

/**
 * Feeds the Platform Blocks theme into React Navigation so navigator-owned
 * surfaces (scene background, headers, the tab bar's defaults) follow the same
 * light/dark scheme as the components instead of Navigation's built-in themes.
 */
function NavigationThemeBridge({ children }: { children: ReactNode }) {
  const theme = useTheme();
  const { actualColorScheme } = useThemeMode();

  const navigationTheme = useMemo(() => {
    const base = actualColorScheme === 'dark' ? DarkTheme : DefaultTheme;
    return {
      ...base,
      colors: {
        ...base.colors,
        primary: theme.colors.primary[6] ?? base.colors.primary,
        background: theme.backgrounds.base,
        card: theme.backgrounds.surface,
        text: theme.text.primary,
        border: theme.backgrounds.border,
      },
    };
  }, [theme, actualColorScheme]);

  return <NavigationThemeProvider value={navigationTheme}>{children}</NavigationThemeProvider>;
}

export default function RootLayout() {
  // Persist the reader's light/dark/auto choice. On web this pairs with the
  // pre-hydration script in +html.tsx (same storage key, same class names) so
  // a saved "dark" applies before first paint.
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
    <SafeAreaProvider>
      <PlatformBlocksProvider themeModeConfig={themeModeConfig}>
        <ContentReveal />
        <StatusBar style="auto" />
        <NavigationThemeBridge>
          <Stack screenOptions={{ headerShown: false }}>
            <Stack.Screen name="(tabs)" />
          </Stack>
        </NavigationThemeBridge>
      </PlatformBlocksProvider>
    </SafeAreaProvider>
  );
}
```

## Flash-free dark mode on static web output — `app/+html.tsx`

From expo-template. Web-only root HTML; the inline script resolves the color
scheme (saved choice first, then OS preference) and stamps it on `<html>` before
first paint; for dark readers it holds content invisible until `ContentReveal`
(above) lifts the class, with a 4 s fallback timer:

```tsx
import { ScrollViewStyleReset } from 'expo-router/html';
import { type PropsWithChildren } from 'react';

/**
 * Web-only root HTML for every statically rendered page.
 *
 * The inline script resolves the reader's colour scheme (saved choice first,
 * then the OS preference) and stamps it on <html> before first paint, so
 * dark-mode readers never see a light flash. The prerendered markup itself
 * carries light-theme styles, so for dark readers the script also holds the
 * content invisible until React has restyled it — ContentReveal in
 * app/_layout.tsx lifts the class, and the timer below is the fallback.
 */
export default function Root({ children }: PropsWithChildren) {
  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta httpEquiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
        <ScrollViewStyleReset />
        <style dangerouslySetInnerHTML={{ __html: baseStyles }} />
        <script dangerouslySetInnerHTML={{ __html: themeScript }} />
      </head>
      <body>{children}</body>
    </html>
  );
}

const baseStyles = `
body {
  background-color: #fff;
  margin: 0;
  padding: 0;
}

/* Auto mode: the provider leaves both theme classes off <html>, so the OS
   preference decides the backdrop. */
@media (prefers-color-scheme: dark) {
  body {
    background-color: #000;
  }
}

/* An explicit light/dark choice beats the OS preference above. The classes are
   stamped by the pre-hydration script below and kept in sync by the provider. */
html.platform-blocks-light,
html.platform-blocks-light body {
  background-color: #fff;
}

html.platform-blocks-dark,
html.platform-blocks-dark body {
  background-color: #000;
}

#root {
  display: flex;
  flex: 1;
  height: 100vh;
  width: 100vw;
}

/* Dark readers: hold the light-styled prerendered content invisible (dark
   backdrop only) until hydration restyles it. Removed by ContentReveal in
   app/_layout.tsx, or by the script's fallback timer. */
html.platform-blocks-content-pending #root {
  visibility: hidden;
}
`;

const themeScript = `
(function() {
  var root = document.documentElement;
  var scheme = 'light';
  try {
    var saved = null;
    try { saved = localStorage.getItem('platform-blocks-theme-mode'); } catch (storageError) {}

    if (saved === 'dark' || saved === 'light') {
      scheme = saved;
    } else {
      scheme = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }

    root.classList.remove('platform-blocks-light', 'platform-blocks-dark');
    root.classList.add('platform-blocks-' + scheme);
    root.style.colorScheme = scheme;
    root.style.backgroundColor = scheme === 'dark' ? '#000000' : '#ffffff';

    if (scheme === 'dark') {
      root.classList.add('platform-blocks-content-pending');
      setTimeout(function () {
        root.classList.remove('platform-blocks-content-pending');
      }, 4000);
    }
  } catch (e) {
    // Leave whatever resolved above in place; never downgrade to light here.
  }
})();
`;
```

## Babel — `babel.config.js`

Identical in both templates:

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    // Worklets plugin (replaces the old reanimated plugin) must be last.
    plugins: ['react-native-worklets/plugin'],
  };
};
```

## Jest config — `package.json`

```json
"jest": {
  "preset": "jest-expo",
  "resolver": "react-native-worklets/jest/resolver.js",
  "transformIgnorePatterns": [
    "/node_modules/(?!(.pnpm|react-native|@react-native|@react-native-community|expo|@expo|@expo-google-fonts|react-navigation|@react-navigation|@sentry/react-native|native-base|standard-navigation|@platform-blocks|@tabler/icons-react-native|@shopify/flash-list|react-native-reanimated-carousel))",
    "/node_modules/react-native-reanimated/plugin/",
    "/node_modules/@react-native/babel-preset/"
  ]
}
```

Remember: react-native must be 0.86.3+ alongside jest-expo (0.86.0 conflicts
with jest-expo's `@react-native/jest-preset` peer).

## Component test — `__tests__/home.test.tsx`

From expo-template. Wrap the tree in `SafeAreaProvider` with explicit
`initialMetrics` (no native module measures insets under Jest), then
`PlatformBlocksProvider`:

```tsx
import { PlatformBlocksProvider } from '@platform-blocks/ui';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import renderer, { act } from 'react-test-renderer';

import HomeScreen from '../app/(tabs)/index';

const TEST_SAFE_AREA_METRICS = {
  frame: { x: 0, y: 0, width: 390, height: 844 },
  insets: { top: 0, left: 0, right: 0, bottom: 0 },
};

describe('HomeScreen', () => {
  it('renders the welcome copy', async () => {
    let tree: renderer.ReactTestRenderer;
    await act(async () => {
      tree = renderer.create(
        <SafeAreaProvider initialMetrics={TEST_SAFE_AREA_METRICS}>
          <PlatformBlocksProvider>
            <HomeScreen />
          </PlatformBlocksProvider>
        </SafeAreaProvider>
      );
    });

    const json = JSON.stringify(tree!.toJSON());
    expect(json).toContain('Hello, Platform Blocks');

    await act(async () => {
      tree!.unmount();
    });
  });
});
```

## Typecheck script (TypeScript ~6 + Node 24)

```json
"typecheck": "node --stack-size=8192 ./node_modules/typescript/lib/_tsc.js --noEmit"
```

Plain `tsc --noEmit` can stack-overflow on expo-router's vendored navigation
types; the deeper stack fixes it. Non-router apps can keep `tsc --noEmit`.

## Verify the install

```tsx
import React from 'react';
import { Text, Button, Card } from '@platform-blocks/ui';

export function TestComponent() {
  return (
    <Card variant='outline'>
      <Text variant='h2'>
        Welcome to PlatformBlocks! 🎉
      </Text>
      <Button
        title='It works!'
        variant='filled'
        onPress={() => console.log('Success!')}
      />
    </Card>
  );
}
```
