# Platform Blocks feedback & overlays — API reference

All symbols import from `@platform-blocks/ui`. Verified against
`packages/ui/src` (Toast, Dialog, Popover, Menu, Tooltip, ContextMenu, Alert,
Overlay, LoadingOverlay, Loader, Progress, Ring, Skeleton,
core/providers/OverlayProvider, hooks/useDisclosure).

## Root exports

```ts
// Toast
Toast, ToastProvider, useToast, useToastApi, toasts,
onToastsRequested, useToastViewportOffset, setToastViewportOffset
type ToastProps, ToastViewportOffset

// Dialog
Dialog, DialogProvider, DialogRenderer, useDialog, useDialogApi, useDialogs,
useSimpleDialog, onDialogsRequested
type DialogProps, DialogConfig, DialogAutoFocus, UseSimpleDialogOptions

// Anchored overlays
Popover                      // Popover.Target, Popover.Dropdown
Menu, MenuDropdown, MenuItem, MenuLabel, MenuDivider, MenuSub
Tooltip, resolveTooltipProps, getTooltipText
ContextMenu
type PopoverProps, PopoverTargetProps, PopoverDropdownProps,
     MenuProps, MenuItemProps, MenuSubProps, TooltipProps, ContextMenuProps

// Inline + loading
Alert, Notice                // Notice is a deprecated alias
Overlay, LoadingOverlay, Loader, Skeleton, Ring
Progress, ProgressRoot, ProgressSection, ProgressLabel

// Overlay engine + state
OverlayProvider, useOverlay, useOverlayApi, useOverlays
useDisclosure, useOverlayMode, useEscapeKey
```

## Provider wiring

`PlatformBlocksProvider` mounts `OverlayProvider` + `OverlayRenderer` when
`withOverlays` is `true` (the default). `ToastProvider` and `DialogProvider` are
**not** included — mount them yourself inside `PlatformBlocksProvider`.

`ToastProviderProps`: `defaultPosition`, `limit` (max toasts per position),
`autoHide` (ms), `defaultVariant`, `defaultSize`, plus a static viewport offset
(a dynamic offset published via `setToastViewportOffset` /
`useToastViewportOffset` wins per-axis).

## useDisclosure

```ts
const [opened, { open, close, toggle }] = useDisclosure(
  initialState = false,
  callbacks?: { onOpen?: () => void; onClose?: () => void },
);
```

Callbacks fire only on real transitions — `open()` on an already-open value is a
no-op and does not call `onOpen`.

## Toast API

`useToast()` (and the standalone `toasts` object) return:

| Method | Signature | Notes |
| --- | --- | --- |
| `show` / `send` | `(options: ToastOptions) => string` | `send` is an alias |
| `info` `success` `warning` `warn` `error` | `(options: ToastShortcut) => string` | `ToastShortcut = string \| Omit<ToastOptions, 'sev'>` |
| `hide` | `(id: string) => void` | |
| `hideAll` | `() => void` | |
| `hideGroup` | `(groupId: string) => void` | |
| `update` | `(id: string, options: Partial<ToastOptions>) => void` | |
| `batch` | `(toasts: ToastOptions[]) => string[]` | |
| `promise` | `<T>(p: Promise<T>, { pending, success, error }) => Promise<T>` | `success`/`error` may be functions of the value/error |

`ToastOptions extends Omit<ToastProps, 'visible' \| 'onClose' \| 'position'>`
plus `id`, `position`, `autoHide`, `message`, `priority`, `groupId`.

### ToastProps

| Prop | Type | Default |
| --- | --- | --- |
| `variant` | `'light' \| 'filled' \| 'outline'` | `'light'` |
| `color` | `'primary' \| 'secondary' \| 'success' \| 'warning' \| 'error' \| 'gray' \| string` | `'gray'` |
| `sev` | `'info' \| 'success' \| 'warning' \| 'error'` | — |
| `size` | `ComponentSizeValue` (`xs`–`3xl` or number) | `'md'` |
| `title` | `string` | — |
| `children` | `ReactNode` | — |
| `icon` | `ReactNode` | — |
| `withCloseButton` | `boolean` | `true` |
| `closeButtonLabel` | `string` | `'Close notification'` |
| `loading` | `boolean` | `false` |
| `autoHide` | `number` (ms, `0` = never) | `4000` |
| `persistent` | `boolean` | `false` |
| `position` | `'top' \| 'bottom' \| 'left' \| 'right'` | `'top'` |
| `actions` | `{ label: string; onPress: () => void; color?: string }[]` | — |
| `dismissOnTap` | `boolean` | `false` |
| `transitionDuration` | `number` (takes precedence over `animationDuration`) | `300` |
| `maxWidth` | `number` | — |
| `keepMounted` | `boolean` | `true` |
| `swipeConfig` / `onSwipeDismiss` | swipe-to-dismiss | — |
| `titleProps` / `bodyProps` | `Omit<TextProps, 'children'>` | — |

Swipe-to-dismiss needs `react-native-gesture-handler`; without it the toast
still renders and the gesture is disabled.

## Dialog

| Prop | Type | Default |
| --- | --- | --- |
| `visible` | `boolean` | **required** |
| `children` | `ReactNode` | **required** |
| `variant` | `'modal' \| 'bottomsheet' \| 'fullscreen'` | — |
| `title` | `string \| null` | — |
| `onClose` | `() => void` | — |
| `closable` | `boolean` | — |
| `backdrop` / `backdropClosable` | `boolean` | — |
| `shouldClose` | `boolean` | triggers the close animation |
| `showHeader` | `boolean` | `true` |
| `w` / `h` / `radius` | `number` | — |
| `transitionDuration` | `number` (`0` = instant; always `0` under reduced motion) | `300` |
| `autoFocus` | `DialogAutoFocus` = `boolean \| RefObject<any>` | `false` |
| `trapFocus` | `boolean` (web only) | `true` |
| `bottomSheetSwipeZone` | `'container' \| 'handle' \| 'none'` | — |
| `titleProps` | `Omit<TextProps, 'children'>` | — |

`useDialog()` → `{ openDialog(config) => id, closeDialog(id), closeAllDialogs() }`.
`useSimpleDialog()` → `{ modal, bottomSheet, fullScreen, confirm, close, closeAll }`,
where the first four take `(content, options?: UseSimpleDialogOptions)` and
`UseSimpleDialogOptions` is `{ title, closable, backdrop, backdropClosable,
width, height, autoFocus, trapFocus }`.

**`confirm()` is unfinished** — its buttons call `closeDialog('')`, which matches
no dialog id, so the prompt never closes. Roll your own (patterns.md).

## Popover

`Popover` + `Popover.Target` + `Popover.Dropdown`.

| Prop | Type | Default |
| --- | --- | --- |
| `opened` / `defaultOpened` | `boolean` | `false` (uncontrolled) |
| `onChange` | `(opened: boolean) => void` | — |
| `onOpen` / `onClose` / `onDismiss` | `() => void` | `onDismiss` = outside click or Escape |
| `position` | `PlacementType` (`'top' \| 'bottom' \| 'left' \| 'right'` + `-start`/`-end`, `'auto'`) | — |
| `trigger` | `'click' \| 'hover'` | `'click'` |
| `withArrow` / `arrowSize` / `arrowRadius` / `arrowOffset` | arrow config | `withArrow` `false` |
| `closeOnClickOutside` / `closeOnEscape` | `boolean` | `true` |
| `trapFocus` / `returnFocus` | `boolean` (web) | `returnFocus` `false` |
| `keepMounted` | `boolean` | `false` |
| `withinPortal` | `boolean` | `true` |
| `withOverlay` / `overlayProps` | backdrop | `false` |
| `w` | `number \| 'target'` | — |
| `maxW` / `minW` / `maxH` / `minH` | `number` | — |
| `radius` / `shadow` / `zIndex` | | `zIndex` `300` |
| `disabled` | `boolean` | `false` |

## Menu

Trigger is `Menu`'s **first child**; the dropdown is `MenuDropdown`. There is no
`Menu.Target` and no `Menu.Item`.

| Prop | Type | Default |
| --- | --- | --- |
| `opened` | `boolean` | — |
| `trigger` | `'click' \| 'hover' \| 'contextmenu'` | `'click'` |
| `position` | same placement union as Popover | `'auto'` |
| `offset` | `number` | `4` |
| `closeOnClickOutside` / `closeOnEscape` | `boolean` | `true` |
| `onOpen` / `onClose` | `() => void` | — |
| `w` | `number \| 'target' \| 'auto'` | `'auto'` |
| `maxH` | `number` | `300` |
| `shadow` / `radius` | `'none' \| 'sm' \| 'md' \| 'lg' \| 'xl'` | `'md'` |
| `strategy` | `'absolute' \| 'fixed' \| 'portal'` | web `'fixed'`, native `'portal'` |
| `disabled` | `boolean` | `false` |

`MenuItemProps`: `children`, `onPress`, `disabled`, `startSection`,
`endSection`, `color` (`'default' \| 'danger' \| 'success' \| 'warning'`),
`closeMenuOnClick`, `testID`, plus spacing props.
`MenuLabel`, `MenuDivider`, `MenuSub` are siblings of `MenuItem`.

## Tooltip

| Prop | Type | Default |
| --- | --- | --- |
| `label` | `ReactNode` | **required** |
| `children` | `React.ReactElement` | **required** — a single element |
| `position` | `TooltipPositionType` | `'top'` |
| `withArrow` | `boolean` | `false` |
| `offset` | `number` | `8` |
| `openDelay` / `closeDelay` | `number` (ms) | `0` |
| `opened` | `boolean` | controlled mode |
| `events` | `{ hover?, focus?, touch? }` | **`{ hover: true, focus: false, touch: true }`** |
| `width` / `maxWidth` | `number` | `maxWidth` `280` |
| `lineClamp` | `number` | — |
| `color` / `radius` | | `radius` `'md'` |
| `labelProps` | `Omit<TextProps, 'children'>` | — |

`multiline` is legacy — labels wrap by default; it only matters with `width`.

## ContextMenu

`children` is a **render prop**:
`(props: { onContextMenu: (e) => void; onPressIn: (e) => void }) => ReactNode`.

`ContextMenuItem` is `{ id: string; label: string; icon?: ReactNode;
disabled?: boolean; danger?: boolean; onSelect?: () => void }` — note `id` is
**required** and the handler is `onSelect`, not `onPress`.

Other props: `items: ContextMenuItem[]` (required), `closeOnSelect` (default
`true`), `longPressDelay` (native), `maxHeight`, `open`,
`position: { x, y }`, `onOpen`, `onClose`, `portalId`, `style`.

## Alert

| Prop | Type | Default |
| --- | --- | --- |
| `variant` | `'light' \| 'filled' \| 'outline' \| 'subtle'` | `'light'` |
| `color` | `'primary' \| 'secondary' \| 'success' \| 'warning' \| 'error' \| 'gray' \| string` | `'primary'` |
| `sev` | `'info' \| 'success' \| 'warning' \| 'error'` | sets color **and** icon |
| `title` | `string` | — |
| `icon` | `ReactNode \| string \| null \| false` | string = `Icon` name; only `false` removes it |
| `withCloseButton` / `closeButtonLabel` / `onClose` | | `withCloseButton` `false` |
| `fullWidth` | `boolean` | `false` |
| `titleProps` / `bodyProps` | `Omit<TextProps, 'children'>` | — |

## Loading, progress, placeholders

**`Loader`** — `size` (`SizeValue`, `'md'`), `color`, `variant`
(`'oval' \| 'dots' \| 'bars'`, `'oval'`), `speed` (ms, `1000`), spacing props.

**`Skeleton`** — `shape` (`'text' \| 'chip' \| 'avatar' \| 'button' \| 'card' \|
'circle' \| 'rectangle' \| 'rounded'`, `'rectangle'`), `w`/`h`, `size`
(overrides w/h), `radius`, `animate` (`true`), `animationDuration` (`1500`),
`colors: [string, string]`.

**`Progress`** — `value` **0–100** (required), `size`, `color`, `radius`,
`striped`, `animate`, `transitionDuration` (`0`), `orientation`
(`'horizontal' \| 'vertical'`; vertical fills bottom-up), `length`
(`number | '\${number}%'`; vertical defaults to 160), `trackColor` (defaults to
`gray[1]`). Compound parts `ProgressRoot` / `ProgressSection` / `ProgressLabel`
build segmented bars.

**`Ring`** — `value` (required), normalized by `min` (`0`) / `max` (`100`);
`size` (`100`), `thickness` (`12`), `label`, `subLabel`, `caption`, `showValue`
(`true`), `valueFormatter(value, percent)`, `trackColor`, `progressColor`
(string or `(value, percent) => string`), `colorStops` (`{ value: number; color: string }[]`, thresholds 0–100), `neutral`,
`roundedCaps` (`true`), `children` as node or `(context) => ReactNode`.

**`Overlay`** — `color`, `opacity` (`0.6`), `backgroundOpacity` (`1`),
`gradient` (web only), `blur` (web), `radius`, `zIndex`, `fixed`, `center`,
`children`.

**`LoadingOverlay`** — `visible` (`false`), `zIndex`, `overlayProps`,
`loaderProps`, `loader` (replaces the `Loader` entirely). Fills its nearest
positioned ancestor.

## OverlayProvider (low level)

`useOverlayApi()` → `{ openOverlay(config) => id, closeOverlay(id),
closeAllOverlays(), updateOverlay(id, updates) }`; `useOverlays()` returns the
live list. `OverlayConfig` covers `content`, `trigger`, `placement`, `offset`,
`anchor`, `anchorNode`, `pinEdge`/`pinOffset`, `width`/`maxWidth`/`maxHeight`,
`closeOnClickOutside`, `closeOnEscape`, `strategy`, `zIndex`, `viewport`.
Prefer `Popover`/`Menu`/`Tooltip` — reach for this only for custom overlays.

`useOverlayMode({ forceModal?, forceOverlay? })` reports whether the current
device should get a modal or an anchored overlay presentation.
`useEscapeKey(handler, enabled = true)` is a thin `useHotkeys` wrapper.
