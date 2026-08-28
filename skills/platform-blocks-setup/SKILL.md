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
2. **Install the dependency set** — on **v1.0.0** that means the full list, not just
   the hard peers; on **v1.0.1+** only what your components need (see pitfall #1).
   The safe default either way is to copy the exact working dependency list from
   `expo-template/package.json` (reproduced in `references/api.md`). On Expo, use
   `npx expo install` so versions match the SDK.
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

1. **How many dependencies you actually need depends on the version.** Check with
   `npm ls @platform-blocks/ui` before deciding.

   - **v1.0.0 — the "optional" peers are mandatory.** Metro statically resolves
     the eager `require()` map in the library's `optionalModule` helper, plus
     static imports in Masonry (`@shopify/flash-list`), Carousel
     (`react-native-reanimated-carousel`), and GradientText/ShimmerText
     (`@react-native-masked-view/masked-view`). The app will not bundle without
     all of: `@shopify/flash-list`, `react-native-reanimated-carousel`,
     `@react-native-masked-view/masked-view`, `expo-clipboard`, `expo-haptics`,
     `expo-linear-gradient`, `expo-document-picker`, `react-native-webview`,
     `lodash.debounce`, `expo-audio`, `react-native-gesture-handler`,
     `expo-status-bar`, `expo-navigation-bar`. Several are not declared as peers
     at all, so `npm install` gives no warning — the failure appears only as a
     Metro "Unable to resolve module" error. Install the whole set up front.

   - **v1.0.1+ — they are genuinely optional.** Every loader `require()` now sits
     inside its own lexical `try/catch` (the shape Metro's
     `allowOptionalDependencies` needs to see at each call site), and Masonry,
     Carousel, DataTable, GradientText and ShimmerText resolve their engines
     lazily. Install only what the components you actually use need; each missing
     module degrades gracefully with a dev warning rather than breaking the
     bundle. See the degradation table in `references/api.md`. The one hard case
     is `Carousel`, which has no engine without
     `react-native-reanimated-carousel` and cannot render at all.

   Installing the full set is always safe on either version — it just costs
   install size on 1.0.1+.

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

## Anything this skill does not cover

This skill covers installing and configuring the library so an app builds,
renders, and tests. Platform Blocks is much larger — 97 components, 25 charts,
and 18 hooks. Do not guess an API for something outside this scope; fetch the
generated docs instead:

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

- **Component and hook APIs** — this skill stops once the app renders; it does
  not document what any component does.
- **Theming** → the `platform-blocks-theming` skill. **Layout and app chrome** →
  `platform-blocks-layout`. **Forms and inputs** → `platform-blocks-forms`.
  **Charts** → `platform-blocks-charts`.
