---
name: plocks-setup
description: Install and configure @plocks/ui in an Expo or React Native app. Use for package setup, PlocksProvider, peer dependencies, Metro or Jest configuration, and web color-scheme bootstrapping.
---

# plocks setup

Use the package manifest for the version being installed as the source of truth for peer dependencies. The library and its starter templates are pre-release; do not infer a package version or compatibility path from the prior brand.

## Choose a starting point

- For an Expo Router app on iOS, Android, and web, start from [expo-template](https://github.com/platform-blocks/expo-template).
- For one screen with minimal setup, start from [expo-min-template](https://github.com/platform-blocks/expo-min-template).
- For other targets, see the [template catalog](https://plocks.dev/getting-started).

A template already wires the provider, native dependencies, Babel, and any platform-specific bootstrapping. For a hand-built app, follow the steps below.

## Manual setup

1. Install `@plocks/ui` and the required peers listed in [references/api.md](references/api.md). In Expo, use `npx expo install` for native dependencies so they match the SDK.
2. With Reanimated 4, install `react-native-worklets` and put `react-native-worklets/plugin` last in the Babel plugin list.
3. Wrap the app in `PlocksProvider`. It mounts the safe-area, overlay, theme, and i18n services by default. Mount `ToastProvider` and `DialogProvider` inside it only if those features are used.
4. Render a `Text`, `Card`, and `Button` from `@plocks/ui` to verify the app and native dependencies resolve.
5. For Expo Router, bridge the plocks theme into React Navigation so navigator-owned headers and tab bars follow the same scheme. See [references/patterns.md](references/patterns.md).

Optional peers are installed when their corresponding feature needs them. Check the package manifest and the specific component before installing an extension. Other plocks packages (`@plocks/charts`, `@plocks/dates`, `@plocks/code`, `@plocks/media`, `@plocks/carousel`, `@plocks/spotlight`, `@plocks/brands`, `@plocks/qrcode`) are separate imports.

## Dark mode

Pass `themeModeConfig={{ initialMode: 'auto' }}` to `PlocksProvider` to expose `useThemeMode()`. Its persistence adapter uses synchronous `get` and `set` methods. On statically rendered web pages, apply the stored mode in `app/+html.tsx` before hydration; use the web template's script as the reference. The default storage key and DOM classes are documented in [references/api.md](references/api.md).

## Testing

Use `jest-expo` for an Expo consumer app. Include `@plocks` in `transformIgnorePatterns` so Jest transforms package code, and use the worklets Jest resolver when Reanimated 4 is installed. Wrap rendered UI in `PlocksProvider`; it supplies zero safe-area insets in tests. See [references/patterns.md](references/patterns.md) for a minimal test.

## When something fails

- `Unable to resolve module`: compare installed dependencies with `peerDependencies` and `peerDependenciesMeta` in the exact package version. A package marked optional may still be necessary for the component being rendered.
- Animation or worklet errors: check the installed Reanimated and Worklets versions and confirm the Worklets Babel plugin is last.
- Web light flash: check that the pre-hydration script reads the same storage key as `themeModeConfig` and stamps the color-scheme marker before React starts.

For a component outside this skill, use the [docs index](https://plocks.dev/llms.txt) and import it from its owning package.
