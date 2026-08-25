---
name: platform-blocks-setup
description: Install and configure the @platform-blocks/ui React Native library in an Expo or React Native app. Use when installing @platform-blocks/ui, wiring PlatformBlocksProvider, fixing peer-dependency or Metro "Unable to resolve module" bundling errors, setting up flash-free dark mode (including static web output), or configuring Jest/jest-expo in a consumer app.
---

# Platform Blocks Setup

Platform Blocks (`@platform-blocks/ui`) is a React Native UI library — 80+ themeable,
accessible components for iOS, Android, and Web. Docs: https://platform-blocks.com/getting-started

## Fastest start: official templates

Prefer a template over manual setup — both ship with the library, all required
dependencies, and the provider already wired:

- **expo-template** — https://github.com/platform-blocks/expo-template — full-featured
  Expo Router app (iOS/Android/web) with dark mode, testing, and linting configured.
- **expo-min-template** — https://github.com/platform-blocks/expo-min-template — single
  screen, provider set up, nothing to delete.

Use GitHub's "Use this template" button, or:

```bash
npx create-expo-app@latest my-app --template https://github.com/platform-blocks/expo-template
```

## Core workflow (manual install)

1. **Install the library**: `npm install @platform-blocks/ui`
2. **Install the full dependency set** — not just the hard peers (see pitfall #1 below).
   Copy the exact working dependency list from `expo-template/package.json`
   (reproduced in `references/api.md`). On Expo, use `npx expo install` so versions
   match the SDK.
3. **Configure Babel** — `presets: ['babel-preset-expo']`,
   `plugins: ['react-native-worklets/plugin']` (the worklets plugin replaces the old
   reanimated plugin and must be **last**).
4. **Wrap the app**: `SafeAreaProvider` > `PlatformBlocksProvider` > your app.
5. **Verify** by rendering a `Card`/`Text`/`Button` from `@platform-blocks/ui`.

## Hard peer dependencies

Required by `peerDependencies` (not marked optional): `react`, `react-native`,
`react-native-reanimated` (v4 needs its companion `react-native-worklets`),
`react-native-safe-area-context`, `react-native-svg`, and
`@tabler/icons-react-native` — the last one backs the Icon registry, which is
imported from the package root, so without it `Icon` and every component that
renders one fails to resolve.

## Common pitfalls

1. **"Optional" peers are required to bundle (v1.0.0).** Metro statically resolves
   the eager `require()` map in the library's `optionalModule` helper, plus static
   imports in Masonry (`@shopify/flash-list`), Carousel
   (`react-native-reanimated-carousel`), and GradientText/ShimmerText
   (`@react-native-masked-view/masked-view`). So the app will not bundle without:
   `@shopify/flash-list`, `react-native-reanimated-carousel`,
   `@react-native-masked-view/masked-view`, `expo-clipboard`, `expo-haptics`,
   `expo-linear-gradient`, `expo-document-picker`, `react-native-webview`,
   `lodash.debounce`, `expo-audio`, `react-native-gesture-handler`,
   `expo-status-bar`, `expo-navigation-bar`. Several of these are not declared as
   peers at all, so `npm install` gives no warning — the failure appears only as a
   Metro "Unable to resolve module" error. Install the whole set up front.
2. **react-native version with Jest.** Use react-native **0.86.3+** when the app
   runs jest-expo — 0.86.0 conflicts with jest-expo's `@react-native/jest-preset`
   peer. (expo-min-template ships 0.86.0 only because it has no Jest setup.)
3. **Worklets plugin ordering.** `react-native-worklets/plugin` must be the last
   Babel plugin. Missing or misplaced, Reanimated-based components crash at runtime.
4. **Missing SafeAreaProvider.** `PlatformBlocksProvider` must sit inside
   `SafeAreaProvider` — in the app root and in every Jest test tree (tests also
   need `initialMetrics`, since there is no native module to measure insets).
5. **TypeScript ~6 + Node 24 stack overflow.** `tsc --noEmit` can overflow the
   default stack on expo-router's vendored navigation types. Use the template's
   typecheck script:
   `node --stack-size=8192 ./node_modules/typescript/lib/_tsc.js --noEmit`
6. **Light flash on statically rendered web.** Prerendered markup carries
   light-theme styles. Fix with the pre-hydration script in `app/+html.tsx` plus
   the `ContentReveal` component in `app/_layout.tsx` (full code in
   `references/patterns.md`).

## Jest in consumer apps

Configure Jest with (full config in `references/api.md`):

- `"preset": "jest-expo"`
- `"resolver": "react-native-worklets/jest/resolver.js"`
- `transformIgnorePatterns` extended with
  `@platform-blocks|@tabler/icons-react-native|@shopify/flash-list|react-native-reanimated-carousel`

Wrap every rendered tree in `SafeAreaProvider` (with `initialMetrics`) and
`PlatformBlocksProvider` — copy the test pattern from `references/patterns.md`.

## Dark mode

`PlatformBlocksProvider` accepts `themeModeConfig` (`ThemeModeConfig`):
`initialMode: 'auto' | 'light' | 'dark'` plus an optional `persistence`
`{ get, set }` pair (the template persists to `localStorage` on web under the key
`platform-blocks-theme-mode`). Read the current scheme with `useThemeMode()`
(`actualColorScheme`) and theme tokens with `useTheme()`. With Expo Router, bridge
the theme into React Navigation via a `NavigationThemeBridge` so navigator-owned
surfaces (headers, tab bar, scene background) follow the same scheme. For static
web output, add the flash-free `+html.tsx` script. All code is in
`references/patterns.md`.

## References

- `references/api.md` — full dependency tables (hard peers, optional-but-required
  modules, exact known-good versions from the templates), Jest config options,
  `ThemeModeConfig` shape, tsconfig, typecheck script, CI workflow.
- `references/patterns.md` — complete copy-paste code: minimal `App.tsx`, Expo
  Router `_layout.tsx` (provider + ThemeModeConfig + ContentReveal +
  NavigationThemeBridge), flash-free `+html.tsx`, `babel.config.js`, Jest test
  with SafeAreaProvider metrics, package.json jest block.
