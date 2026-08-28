# Platform Blocks navigation — API reference

All symbols import from `@platform-blocks/ui` unless noted. Verified against
`packages/ui/src` (Tabs, Breadcrumbs, Pagination, Stepper, TableOfContents,
Spotlight, Link, MenuItemButton, hooks/useHotkeys, hooks/useScrollSpy).

## Root exports

```ts
Tabs                          // items-driven, renders its own content panels
Breadcrumbs
Pagination
Stepper                       // Stepper.Step, Stepper.Completed
TableOfContents               // web only
Spotlight, SpotlightProvider, useSpotlightStore, useSpotlightStoreInstance,
  spotlight, createSpotlightStore, useDirectSpotlightState, directSpotlight,
  onSpotlightRequested
Link, MenuItemButton
useScrollSpy, useHotkeys, useGlobalHotkeys, useEscapeKey, useSpotlightToggle
```

**Subpath only** — `@platform-blocks/ui/Navigation`:
`NavigationContainer`, `Screen`, `createStackNavigator`,
`createDrawerNavigator`, `useNavigation`, `useRoute`.

## Tabs

`TabItem`:

```ts
interface TabItem {
  key: string;                      // returned by callbacks; used for persistence
  label: string | ReactNode;
  subLabel?: string | ReactNode;    // second line under the label
  content: ReactNode;               // REQUIRED unless navigationOnly is set
  disabled?: boolean;
  icon?: ReactNode;
}
```

| Prop | Type | Default |
| --- | --- | --- |
| `items` | `TabItem[]` | **required** |
| `activeTab` | `string` | uncontrolled — first item wins |
| `onTabChange` | `(tabKey: string) => void` | fires in both modes |
| `onDisabledTabPress` | `(tabKey, item) => void` | |
| `variant` | `'line' \| 'chip' \| 'card' \| 'folder'` | `'line'` |
| `size` | `SizeValue` | `'sm'` |
| `color` | `'primary' \| 'secondary' \| 'gray' \| 'tertiary' \| string` | `'primary'` |
| `orientation` | `'horizontal' \| 'vertical'` | `'horizontal'` |
| `location` | `'start' \| 'end'` | `'start'` |
| `scrollable` | `boolean` | `false` |
| `animated` / `animationDuration` | `boolean` / `number` | `true` / `250` |
| `transitionDuration` | `number` — takes precedence; `0` = instant, always 0 under reduced motion | `250` |
| `disabledKeys` | `string[]` | `[]` |
| `navigationOnly` | `boolean` — strip only, `content` ignored | `false` |
| `persistKey` / `autoPersist` | `string` / `boolean` | persists the active tab |
| `tabGap`, `tabCornerRadius`, `contentCornerRadius`, `indicatorThickness`, `activeTabBackgroundColor` | | visual tuning |
| `style` / `tabStyle` / `contentStyle` / `textStyle` / `labelProps` | | |

## Breadcrumbs

`BreadcrumbItem`: `{ label: string; href?: string; icon?: ReactNode;
onPress?: () => void }`. The last item normally has neither `href` nor
`onPress` — that is what marks it as the current page.

| Prop | Type | Default |
| --- | --- | --- |
| `items` | `BreadcrumbItem[]` | **required** |
| `separator` | `ReactNode` | `'/'` |
| `maxItems` | `number` | collapses the **middle**; first and last always shown |
| `size` | `ComponentSizeValue` | `'md'` |
| `showIcons` | `boolean` | `true` |
| `accessibilityLabel` | `string` | `'Breadcrumb navigation'` |
| `style` / `textStyle` / `separatorStyle` / `labelProps` / `separatorProps` | | |

## Pagination

| Prop | Type | Default |
| --- | --- | --- |
| `current` | `number` — **1-indexed** | **required** |
| `total` | `number` — **page count**, not row count | **required** |
| `onChange` | `(page: number) => void` | **required** |
| `siblings` | `number` | `1` |
| `boundaries` | `number` | `1` |
| `size` | `ComponentSizeValue` | `'md'` |
| `variant` | `'default' \| 'outline' \| 'subtle'` | `'default'` |
| `color` | `'primary' \| 'secondary' \| 'gray'` | `'primary'` |
| `showFirst` / `showPrevNext` | `boolean` | `true` |
| `labels` | `{ first?, previous?, next?, last? }` | `{}` |
| `hideOnSinglePage` | `boolean` | `false` |
| `showSizeChanger` | `boolean` | `false` |
| `pageSizeOptions` | `number[]` | `[10, 20, 50, 100]` |
| `pageSize` / `onPageSizeChange` | `number` / `(size) => void` | `10` |
| `showTotal` | `boolean \| ((total, range: [number, number]) => ReactNode)` | `false` |
| `totalItems` | `number` — the **row** count, feeds `showTotal` only | |
| `disabled` | `boolean` | `false` |
| `style` / `buttonStyle` / `activeButtonStyle` / `textStyle` / `activeTextStyle` | | |

`DataTable` already renders a `Pagination` in its footer — configure that one
through the table's `paginationProps`.

## Stepper

| Prop | Type | Notes |
| --- | --- | --- |
| `active` | `number` — **0-indexed** | **required** |
| `children` | `Stepper.Step` / `Stepper.Completed` | **required** |
| `onStepClick` | `(stepIndex: number) => void` | |
| `orientation` | `'horizontal' \| 'vertical'` | |
| `iconPosition` | `'left' \| 'right'` | |
| `iconSize` | `number` | |
| `size` | `ComponentSizeValue` | |
| `color` | `string` | |
| `completedIcon` | `ReactNode` | global default |
| `allowNextStepsSelect` | `boolean` | can users jump ahead |

`Stepper.Step`: `label`, `description`, `icon`, `completedIcon`,
`allowStepSelect`, `color`, `children`.
`Stepper.Completed`: content shown once `active` passes the last step.

## Spotlight

`SpotlightActionData`: `{ id: string; label: string; description?: string;
keywords?: string[]; icon?: string | ReactNode; onPress?: () => void }`.
`icon` accepts an `Icon` registry name string.

| Prop | Type | Default |
| --- | --- | --- |
| `actions` | `SpotlightItem[]` | **required** |
| `shortcut` | `string \| string[] \| null` | **`['cmd+k', 'ctrl+k']`** |
| `nothingFound` | `string` | |
| `highlightQuery` | `boolean \| highlight` | |
| `limit` | `number` | |
| `scrollable` / `maxHeight` | | |
| `variant` | `'modal' \| 'bottomsheet' \| 'fullscreen'` | `'modal'` |
| `width` / `height` | `number` | `width` `600` |
| `searchProps` | | forwarded to the search input |
| `store` | | a store from `createSpotlightStore()` |

Control it from anywhere with the module-level helpers — no hook needed:

```ts
spotlight.open(); spotlight.close(); spotlight.toggle(); spotlight.setQuery('x');
```

`directSpotlight` is the provider-free equivalent (`open`, `close`, `toggle`,
`setQuery`, `getState`), paired with `useDirectSpotlightState()`.
`useSpotlightToggle(handler)` registers `mod+k` globally.

## Link

| Prop | Type | Default |
| --- | --- | --- |
| `children` | `ReactNode` | **required** |
| `href` | `string` | a real URL — not a router route |
| `onPress` | `() => void` | |
| `size` | `SizeValue` | |
| `color` | `'primary' \| 'secondary' \| 'success' \| 'warning' \| 'error' \| 'gray' \| 'inherit' \| string` | `'primary'` |
| `variant` | `'default' \| 'subtle' \| 'hover-underline'` | `'default'` |
| `external` | `boolean` | |
| `target` | `'_blank' \| '_self'` | |
| `disabled` | `boolean` | |
| `accessibilityLabel`, `style`, `textStyle`, `fontFamily` / `ff` | | |

## TableOfContents (web only)

Built on `useScrollSpy` → `IntersectionObserver` + DOM headings. On native it
finds nothing.

| Prop | Type | Default |
| --- | --- | --- |
| `variant` | `'filled' \| 'outline' \| 'ghost' \| 'none'` | `'none'` |
| `color` | `string` | filled variant background |
| `size` | `SizeValue` | `'sm'` |
| `radius` | | |
| `container` | `string \| HTMLElement` | `'main, [role="main"], .main-content, #main-content, article, .content, #content'` |
| `scrollSpyOptions` | `ScrollSpyOptions` | |
| `initialData` | `TocItem[]` | for SSR / prerender |
| `minDepthToOffset` | `number` | `1` |
| `depthOffset` | `number` (px per level) | `20` |
| `autoContrast` | `boolean` | `false` |
| `getControlProps` | `({ data, active, index }) => any` | |
| `onActiveChange` | `(id: string \| null, item?: TocItem) => void` | |
| `reinitializeRef` | `RefObject<() => void>` | manual refresh |

`ScrollSpyOptions`: `selector`, `rootMargin`, `container`, `getDepth(el)`,
`getValue(el)`.
`useScrollSpy(options?, initialData = [])` returns the items plus `activeId`.

## Hotkeys

```ts
type HotkeyItem = [
  string,                              // 'mod+k', 'escape', 'ctrl+j' — case-insensitive
  (event: KeyboardEvent) => void,
  KeyboardModifiers?,                  // optional modifier override
  string[]?,                           // optional description
];

useHotkeys(hotkeys: HotkeyItem[], dependencies?: React.DependencyList): void
useGlobalHotkeys(id: string, hotkey: HotkeyItem): void
useEscapeKey(handler: () => void, enabled = true): void
```

Modifiers: `mod` (⌘ on Mac, Ctrl elsewhere), `ctrl`, `alt`, `meta`/`cmd`,
`shift`. Hotkey strings are lowercased before parsing, so `'mod+K'` and
`'mod+k'` behave identically.

## Expo Router: which component to use

| You want | Use |
| --- | --- |
| Bottom tab bar with real routes | `Tabs` from **`expo-router`**, under `app/(tabs)/` |
| Stack / drawer routing | `expo-router` (`Stack`, `Drawer`) |
| Tabs inside one screen | `Tabs` from **`@platform-blocks/ui`** |
| App chrome (header / navbar / aside) | `AppShell` — see the `platform-blocks-layout` skill |
| An in-app link that is also a real `<a>` on web | the `RouteLink` pattern in `patterns.md` |

`@platform-blocks/ui/Navigation` is a self-contained navigator for apps that do
**not** use Expo Router. Do not mix the two.
