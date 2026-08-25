# Platform Blocks setup — API and dependency reference

Verified against `@platform-blocks/ui@1.0.0` and the official templates
(`platform-blocks/expo-template`, `platform-blocks/expo-min-template`).
Docs: https://platform-blocks.com/getting-started

## Install commands

```bash
npm install @platform-blocks/ui

# Hard peers — on Expo, use expo install so versions match the SDK:
npx expo install react-native-reanimated react-native-safe-area-context react-native-svg @tabler/icons-react-native

# without Expo
npm install react-native-reanimated react-native-safe-area-context react-native-svg @tabler/icons-react-native
```

Then install the full "optional-but-required" set below (see the Metro section for why).

## Hard peer dependencies

Declared in `peerDependencies` and NOT marked optional in `peerDependenciesMeta`:

| Package | Range (peerDependencies) | Notes |
| --- | --- | --- |
| `react` | `>=18.0.0` | |
| `react-native` | `>=0.73.0` | Use **0.86.3+** with jest-expo (see below) |
| `react-native-reanimated` | `>=3.4.0` | v4 requires `react-native-worklets` |
| `react-native-safe-area-context` | `>=4.5.0` | `SafeAreaProvider` must wrap the provider |
| `react-native-svg` | `>=13.0.0` | |
| `@tabler/icons-react-native` | `>=3.0.0` | Backs the Icon registry, imported from the package root — without it `Icon` and every component that renders one fails to resolve |

`react-native-worklets` (`>=0.5.0`) is listed as an optional peer, but it is the
Reanimated 4 companion and supplies the Babel plugin and Jest resolver — treat it
as required in any Reanimated 4 app.

## Optional peers that are REQUIRED to bundle (v1.0.0)

Metro statically resolves every `require()` in the library's eager
`optionalModule` loader map (`packages/ui/src/utils/optionalModule.ts` — "Metro
bundler requires static string literals for require; keep all optional modules
here"), plus static top-of-file imports in `Masonry` (`@shopify/flash-list`),
`Carousel` (`react-native-reanimated-carousel`), and `GradientText`/`ShimmerText`
(`@react-native-masked-view/masked-view`). The app will not bundle without all of:

| Package | Declared as peer? | Why Metro needs it |
| --- | --- | --- |
| `@shopify/flash-list` | optional peer | Static import in Masonry + loader map |
| `react-native-reanimated-carousel` | optional peer | Static import in Carousel |
| `@react-native-masked-view/masked-view` | optional peer | Static import in GradientText/ShimmerText |
| `expo-clipboard` | **not declared** | Eager loader map |
| `expo-haptics` | optional peer | Eager loader map |
| `expo-linear-gradient` | optional peer | Eager loader map |
| `expo-document-picker` | optional peer | Eager loader map |
| `react-native-webview` | **not declared** | Eager loader map |
| `lodash.debounce` | **not declared** | Eager loader map |
| `expo-audio` | optional peer | Eager loader map |
| `react-native-gesture-handler` | **not declared** | Eager loader map |
| `expo-status-bar` | optional peer | Eager loader map |
| `expo-navigation-bar` | **not declared** | Eager loader map |

The "not declared" rows produce no npm peer warning — the only symptom of a
missing one is a Metro `Unable to resolve module` error at bundle time.

`react-syntax-highlighter` (optional peer, `>=15.0.0`) is deliberately NOT in the
loader map (it breaks Metro on native when absent); it is only needed for
web-only syntax highlighting.

## Known-good dependency set (from expo-template, Expo SDK 57)

Copy this exact set from `expo-template/package.json`:

```json
"dependencies": {
  "@platform-blocks/ui": "^1.0.0",
  "@react-native-masked-view/masked-view": "^0.3.2",
  "@shopify/flash-list": "^2.3.1",
  "@tabler/icons-react-native": "^3.46.0",
  "expo": "~57.0.8",
  "expo-audio": "~57.0.4",
  "expo-clipboard": "~57.0.1",
  "expo-constants": "~57.0.0",
  "expo-document-picker": "~57.0.1",
  "expo-haptics": "~57.0.1",
  "expo-linear-gradient": "~57.0.1",
  "expo-linking": "~57.0.0",
  "expo-navigation-bar": "~57.0.2",
  "expo-router": "~57.0.0",
  "expo-status-bar": "~57.0.0",
  "lodash.debounce": "^4.0.8",
  "react": "19.2.3",
  "react-dom": "19.2.3",
  "react-native": "0.86.3",
  "react-native-gesture-handler": "~2.32.0",
  "react-native-reanimated": "~4.5.0",
  "react-native-reanimated-carousel": "^4.0.3",
  "react-native-safe-area-context": "~5.7.0",
  "react-native-screens": "~4.26.0",
  "react-native-svg": "15.15.4",
  "react-native-web": "~0.21.0",
  "react-native-webview": "13.16.1",
  "react-native-worklets": "0.10.0"
},
"devDependencies": {
  "@babel/core": "^7.25.2",
  "@types/jest": "^30.0.0",
  "@types/react": "~19.2.2",
  "eslint": "^8.57.1",
  "eslint-config-expo": "~57.0.0",
  "jest": "~29.7.0",
  "jest-expo": "~57.0.0",
  "react-test-renderer": "19.2.3",
  "typescript": "~6.0.3"
}
```

Notes:

- `expo-constants`, `expo-linking`, `expo-router`, `react-native-screens` are for
  Expo Router; a non-router app (see expo-min-template) omits them.
- **react-native 0.86.3+ with Jest**: 0.86.0 conflicts with jest-expo's
  `@react-native/jest-preset` peer. expo-min-template pins 0.86.0 only because it
  has no Jest setup; add Jest and you must bump to 0.86.3+.

## Jest configuration (consumer apps)

From `expo-template/package.json`:

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

Key points:

- `preset: "jest-expo"` — the Expo-aware preset.
- `resolver: "react-native-worklets/jest/resolver.js"` — required for
  Reanimated 4 / worklets to resolve under Jest.
- The `transformIgnorePatterns` allowlist must include `@platform-blocks`,
  `@tabler/icons-react-native`, `@shopify/flash-list`, and
  `react-native-reanimated-carousel` so those untranspiled packages get Babel-ed.
- Test trees must be wrapped in `SafeAreaProvider` with `initialMetrics`
  (code in `references/patterns.md`).

## Babel configuration

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

## TypeScript

`expo-template/tsconfig.json`:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "types": ["jest"]
  }
}
```

**TypeScript ~6 + Node 24 stack overflow**: `tsc --noEmit` can stack-overflow on
expo-router's vendored navigation types. Use the template's script:

```json
"typecheck": "node --stack-size=8192 ./node_modules/typescript/lib/_tsc.js --noEmit"
```

(Non-router apps like expo-min-template can use plain `tsc --noEmit`.)

## Provider config options

`PlatformBlocksProvider` props used in setup:

- `themeModeConfig?: ThemeModeConfig`
  - `initialMode: 'auto' | 'light' | 'dark'`
  - `persistence?: { get: () => 'light' | 'dark' | 'auto' | null; set: (mode) => void }`
    — the template persists to `localStorage` on web under the key
    `platform-blocks-theme-mode` (the same key the `+html.tsx` pre-hydration
    script reads).

Hooks:

- `useThemeMode()` — returns `actualColorScheme` (`'light' | 'dark'`) among others.
- `useTheme()` — theme tokens, e.g. `theme.colors.primary[6]`,
  `theme.backgrounds.base`, `theme.backgrounds.surface`, `theme.backgrounds.border`,
  `theme.text.primary`.

Web dark-mode class contract (provider + `+html.tsx` script keep in sync):
`html.platform-blocks-light`, `html.platform-blocks-dark`, and
`html.platform-blocks-content-pending` (holds `#root` invisible until
`ContentReveal` or a 4 s fallback timer removes it).

## CI reference (expo-template/.github/workflows/ci.yml)

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: 22
  - run: npm install
  - run: npm run lint
  # tsc needs a deeper stack than Node's default for expo-router's
  # vendored navigation types under TypeScript 6 — npm run typecheck
  # wraps it with --stack-size.
  - run: npm run typecheck
  - run: npm test -- --ci
  - run: npx expo export --platform web
```

The template also runs a weekly lockfile-free install (cron `0 6 * * 1`) as a
freshness check against new `@platform-blocks/ui` releases.

## Templates

- https://github.com/platform-blocks/expo-template — Expo Router, iOS/Android/web,
  dark mode + testing + linting wired up.
- https://github.com/platform-blocks/expo-min-template — minimal single screen.

Use GitHub's "Use this template", or
`npx create-expo-app@latest my-app --template <repo-url>`.
