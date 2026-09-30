# Setup reference

The source package manifest is authoritative for ranges and optional peers. The table below summarizes the current `@plocks/ui` contract; use `npx expo install` to choose SDK-compatible native versions.

## Required peers

| Package | Why |
| --- | --- |
| `react`, `react-native` | Runtime and renderer |
| `react-native-reanimated` | Animation engine |
| `react-native-safe-area-context` | Inset-aware overlays and layout |
| `react-native-svg` | Icons and visual components |
| `@tabler/icons-react-native` | Built-in icon registry |

Reanimated 4 also needs `react-native-worklets` and the Worklets Babel plugin, last in the plugin list.

## Optional peers

The current manifest marks these optional: `@react-native-async-storage/async-storage`, `@react-native-masked-view/masked-view`, `@shopify/flash-list`, `expo-clipboard`, `expo-document-picker`, `expo-haptics`, `expo-linear-gradient`, `expo-navigation-bar`, `expo-status-bar`, `react-native-gesture-handler`, and `react-native-worklets`. Install only those needed by the components and platforms in the app. For example, FlashList supplies virtualization, masked-view supplies native gradient text, and Expo document picker supplies native file selection.

## Provider

```tsx
import { PlocksProvider } from '@plocks/ui';

<PlocksProvider themeModeConfig={{ initialMode: 'auto' }}>
  <App />
</PlocksProvider>
```

The root provider supplies the theme, overlays, i18n, safe area, haptics, direction, and reduced-motion context. `withSafeAreaProvider` defaults to `true`. A nested `PlocksProvider` scopes a theme to its subtree without remounting app-level services. `ToastProvider` and `DialogProvider` are opt-in and belong inside the root provider.

`ThemeModeConfig` accepts an initial mode (`'auto' | 'light' | 'dark'`) and optional synchronous persistence `{ get, set }`. On web, the default storage key is `plocks-theme-mode`; the resolved scheme is stamped as `data-plocks-color-scheme` on `<html>`, and manual choices use `plocks-light` or `plocks-dark` classes. Keep the pre-hydration script consistent with these names.

## Consumer Jest configuration

```json
{
  "preset": "jest-expo",
  "resolver": "react-native-worklets/jest/resolver.js",
  "transformIgnorePatterns": [
    "/node_modules/(?!(.pnpm|react-native|@react-native|@react-native-community|expo|@expo|@plocks|@tabler/icons-react-native))"
  ]
}
```

Adapt the transform allowlist to the native dependencies used by the app. Keep the Worklets resolver when using Reanimated 4.

## Source references

- [UI manifest](https://github.com/platform-blocks/plocks/blob/HEAD/packages/ui/package.json)
- [Expo Router template](https://github.com/platform-blocks/expo-template)
- [Minimal Expo template](https://github.com/platform-blocks/expo-min-template)
- [Getting started](https://plocks.dev/getting-started)
