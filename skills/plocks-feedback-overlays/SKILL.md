---
name: plocks-feedback-overlays
description: Show feedback and overlays with the @plocks/ui React Native library. Use when raising notifications with Toast/ToastProvider/useToast, opening modals, bottom sheets or confirm prompts with Dialog/DialogProvider/useDialog/useSimpleDialog, attaching anchored overlays (Popover, Menu/MenuDropdown/MenuItem, Tooltip, ContextMenu), rendering inline messages with Alert, or showing loading and progress state (Loader, LoadingOverlay, Overlay, Progress, Ring, Skeleton), including open-state management with useDisclosure.
---

# plocks Feedback & Overlays

Everything that appears *over* or *about* the current screen in
`@plocks/ui` — notifications, modals, anchored dropdowns, inline
messages, and loading state. All imports are from the package root:

```tsx
import {
  ToastProvider, useToast, Toast,
  Dialog, DialogProvider, useDialog, useSimpleDialog,
  Popover, Menu, MenuDropdown, MenuItem, Tooltip, ContextMenu,
  Alert, Overlay, LoadingOverlay, Loader, Progress, Ring, Skeleton,
  useDisclosure,
} from '@plocks/ui';
```

Four reference files sit alongside this one:

- `references/api.md` — the curated API: how the pieces compose, the union
  types, the defaults that bite. **Read this first.**
- `references/props.md` — generated, exhaustive prop tables for every component
  in this skill. Look here for the complete surface of a single prop.
- `references/icons.md` — generated, every `name` the built-in `Icon` registry
  accepts. Check it before writing `<Icon name="…">`; unlisted names render
  nothing.
- `references/patterns.md` — complete copy-paste screens.

## Pick the right component

| Need | Use |
| --- | --- |
| Transient "it worked" / "it failed" message | `useToast()` |
| Blocking decision or a form over the screen | `Dialog` (or `useSimpleDialog`) |
| Persistent message *inside* the page flow | `Alert` |
| Rich content anchored to a trigger | `Popover` |
| A list of actions anchored to a trigger | `Menu` |
| Short text hint on hover/focus | `Tooltip` |
| Right-click / long-press actions | `ContextMenu` |
| Block a region while it loads | `LoadingOverlay` |
| Placeholder while content loads | `Skeleton` |
| Determinate progress | `Progress` (bar) or `Ring` (circular) |
| Indeterminate spinner | `Loader` |
| A bare dimming layer you position yourself | `Overlay` |

## Providers — what is and is not mounted for you

`PlocksProvider` mounts `OverlayProvider` + `OverlayRenderer` for you
(controlled by `withOverlays`, default `true`). So **`Popover`, `Menu`,
`Tooltip` and `ContextMenu` work with no extra setup.**

`ToastProvider` and `DialogProvider` are **not** mounted automatically. Add the
ones you use, inside `PlocksProvider`:

```tsx
<PlocksProvider>
  <ToastProvider>
    <DialogProvider>
      <App />
    </DialogProvider>
  </ToastProvider>
</PlocksProvider>
```

## Toast

`useToast()` returns the toast API. Every method returns the toast's `id`:

```tsx
const toast = useToast();

toast.success('Saved');                       // string shortcut
toast.error({ title: 'Upload failed', message: 'Try again.' });
toast.info('Synced');                         // also: warning, warn (alias), info

const id = toast.show({ title: 'Custom', severity: 'info', autoHide: 0 });
toast.update(id, { title: 'Updated' });
toast.hide(id);
toast.hideAll();
```

`toast.promise(promise, { pending, success, error })` drives one toast through a
promise's lifecycle; `success` and `error` may be functions of the resolved
value / thrown error. `toast.batch([...])` shows several at once and returns
their ids; `groupId` + `toast.hideGroup(groupId)` dismisses a set together.

Key `ToastOptions`: `title`, `message` (or `children`), `sev`
(`'info' | 'success' | 'warning' | 'error'`), `variant`
(`'light' | 'filled' | 'outline'`), `color`, `autoHide` (ms, `0` = never),
`persistent`, `position` (`'top' | 'bottom' | 'left' | 'right'`),
`actions: { label, onPress, color }[]`, `dismissOnTap`, `priority`, `groupId`.

`ToastProvider` sets the defaults: `defaultPosition`, `limit` (max per
position), `autoHide`, `defaultVariant`, `defaultSize`.

**Outside React** — there is **no public module-level toast API**. `useToast()`
is the only supported entry point. To raise toasts from a service module or an
HTTP interceptor, capture the API once from inside the tree and export a
module-level handle — full pattern in `references/patterns.md`.

## Dialog

Two ways to open one.

**1. Controlled component** — you own the state. `visible` is required.

```tsx
const [opened, { open, close }] = useDisclosure(false);

<Dialog opened={opened} onClose={close} title="Delete project" variant="modal">
  <Text>This cannot be undone.</Text>
</Dialog>
```

`variant`: `'modal'` | `'bottomsheet'` | `'fullscreen'`. Other useful props:
`closable`, `backdrop`, `backdropClosable`, `showHeader`, `w`/`h`, `radius`,
`transitionDuration` (`0` = instant), `autoFocus`, `trapFocus` (web, default
`true`), `bottomSheetSwipeZone` (`'container' | 'handle' | 'none'`).

**2. Imperative** — needs `DialogProvider`. `useDialog()` gives
`openDialog(config) => id`, `closeDialog(id)`, `closeAllDialogs()`.
`useSimpleDialog()` wraps it with `modal`, `bottomSheet`, `fullScreen`,
`confirm`, `close`, `closeAll`:

```tsx
const dialog = useSimpleDialog();
const id = dialog.modal(<Text>Body</Text>, { title: 'Details' });
dialog.close(id);
```

See pitfall 4 before using `dialog.confirm()`.

## Anchored overlays — note the two different shapes

`Popover` is a **namespace** compound and needs an explicit `Popover.Target`.
`Menu` uses **flat sibling exports** and treats its *first child* as the
trigger — there is no `Menu.Target`, and no `Menu.Item`:

```tsx
// Popover — namespaced, explicit target
<Popover position="bottom" withArrow>
  <Popover.Target><Button>Open</Button></Popover.Target>
  <Popover.Dropdown><Text>Anything</Text></Popover.Dropdown>
</Popover>

// Menu — flat exports, first child is the trigger
import { Menu, MenuDropdown, MenuItem, MenuDivider, MenuLabel } from '@plocks/ui';

<Menu>
  <Button variant="outline">Actions</Button>
  <MenuDropdown>
    <MenuLabel>Account</MenuLabel>
    <MenuItem startSection={<Icon name="user" size="sm" />}>Profile</MenuItem>
    <MenuDivider />
    <MenuItem color="danger" onPress={signOut}>Log out</MenuItem>
  </MenuDropdown>
</Menu>
```

`Tooltip` wraps a single element and takes `label`:

```tsx
<Tooltip label="Copy to clipboard" position="top" withArrow openDelay={300}>
  <IconButton icon="copy" onPress={copy} />
</Tooltip>
```

`ContextMenu` is a **render-prop**, not a wrapper — it hands you the handlers to
spread onto your trigger:

```tsx
<ContextMenu items={[{ id: 'rename', label: 'Rename', onSelect: rename }]}>
  {({ onContextMenu, onPressIn }) => (
    <Pressable onContextMenu={onContextMenu} onPressIn={onPressIn}>
      <Text>Right-click me</Text>
    </Pressable>
  )}
</ContextMenu>
```

## Alert

Inline, in-flow message — not an overlay. `sev` sets color *and* icon together:

```tsx
<Alert severity="error" title="Payment failed" withCloseButton onClose={dismiss}>
  Your card was declined.
</Alert>
```

`variant`: `'light'` (default) | `'filled'` | `'outline'` | `'subtle'`.
`icon` accepts a node or an `Icon` registry name string. Only `icon={false}`
removes it — `null`/`undefined` falls back to the `sev` icon.

## Loading and progress

```tsx
<Loader size="md" variant="oval" />              {/* 'oval' | 'dots' | 'bars' */}
<Skeleton shape="text" w="80%" />                {/* text|chip|avatar|button|card|circle|rectangle|rounded */}
<Progress value={62} striped animate />          {/* 0–100; orientation="vertical" fills bottom-up */}
<Ring value={62} size={120} thickness={12} />    {/* circular; showValue prints the % */}

<Block style={{ position: 'relative' }}>
  <LoadingOverlay visible={isLoading} />
  {content}
</Block>
```

`LoadingOverlay` fills its **nearest positioned ancestor** — give that container
`position: 'relative'`. Forward props with `overlayProps` / `loaderProps`, or
replace the spinner entirely with `loader={<YourThing />}`.

## Accessibility

- Toasts announce automatically; still give `actions` real labels.
- `Dialog` traps Tab focus on web (`trapFocus`, default `true`) and restores
  focus to the trigger on close. Use `autoFocus` to move focus in on open.
- `Popover`/`Menu`/`Tooltip`/`ContextMenu` close on Escape by default
  (`closeOnEscape`); `Popover` also takes `trapFocus` and `returnFocus`.
- `Tooltip` alone is never an accessible name — put `accessibilityLabel` on the
  trigger too, and enable `events.focus` (off by default) so keyboard users can
  see it at all.
- Set `accessibilityLabel` on `Loader`/`Progress` regions that convey status.

## Pitfalls (verified against source)

1. **`ToastProvider` and `DialogProvider` are not mounted by
   `PlocksProvider`** — only `OverlayProvider` is. Without them
   `useToast()` falls back to the module-level queue (calls are buffered and
   never rendered) and `useDialog()` has nothing to render into.
2. **`Menu` has no `Menu.Item` / `Menu.Target`.** Import `MenuItem`,
   `MenuDropdown`, `MenuDivider`, `MenuLabel`, `MenuSub` as separate named
   exports, and pass the trigger as `Menu`'s first child. `Popover` is the
   opposite — `Popover.Target` and `Popover.Dropdown` are namespaced and the
   target is required.
3. **`Dialog`'s `visible` prop is required** and it is fully controlled — it
   will not open from an internal default. Pair it with `useDisclosure()`.
4. **`useSimpleDialog().confirm()` does not close itself.** Its built-in buttons
   call `closeDialog('')` with an empty id (an unfinished TODO in the source),
   which matches no dialog, so the prompt stays open after Confirm or Cancel.
   The callbacks *do* fire. Build confirms with `openDialog` and your own
   buttons, capturing the returned id — see `references/patterns.md`.
5. **`ContextMenu` takes a render function, not children.** Passing an element
   renders nothing.
6. **`LoadingOverlay` needs a positioned ancestor.** Without
   `position: 'relative'` on the wrapper it covers the wrong box.
7. **`Progress` `value` is 0–100, not 0–1.** `Ring` normalizes against
   `min`/`max` (0/100 by default) instead.
8. **`Toast`'s `autoHide` is milliseconds; `0` disables it** (it does not mean
   "hide immediately"). Use `persistent` for toasts the user must dismiss.
9. **`toasts.promise()` silently no-ops without a provider** — unlike the other
   standalone methods it is not queued; it just returns the promise untouched.
10. **`Tooltip` does not open on keyboard focus by default.** The `events`
    default is `{ hover: true, focus: false, touch: true }` — keyboard users get
    nothing. Pass `events={{ hover: true, focus: true, touch: true }}` on any
    tooltip carrying information, and give the trigger a real
    `accessibilityLabel` regardless. `Tooltip` also needs a single element child
    that can take a ref.

## Anything this skill does not cover

This skill covers notifications, modals, anchored overlays, inline messages, and
loading state. plocks is much larger — 97 components, 25 charts, and 18
hooks. Do not guess an API for something outside this scope; fetch the generated
docs instead:

| What you need | Where |
| --- | --- |
| Index of every page, one line each | `https://plocks.dev/llms.txt` |
| One component or chart | `https://plocks.dev/llms/components/<Name>.md` |
| One hook | `https://plocks.dev/llms/hooks/<useName>.md` |
| Guides | `https://plocks.dev/llms/guides/{getting-started,accessibility,localization}.md` |
| Everything in one file (~1.3 MB) | `https://plocks.dev/llms-full.txt` |

`<Name>` is the exact PascalCase export name — `.../llms/components/DataTable.md`,
`.../llms/components/AreaChart.md`. Each page carries the component's full prop
table (type, required, default, description) plus runnable examples, generated
from the source, so it is authoritative where memory is not. When you are unsure
whether something exists or what it is called, read `llms.txt` first — it lists
every page with a one-line summary.

Import components from their owning package: `@plocks/ui` for core UI,
`@plocks/charts`, `@plocks/dates`, `@plocks/code`, `@plocks/media`,
`@plocks/qrcode`, and `@plocks/spotlight` for extensions. `FormLayout` is
also exported from `@plocks/ui`; the `Navigation` module uses the
`@plocks/ui/Navigation` subpath. Check the package barrel before importing.

Notably outside this skill:

- **Navigation components that also use overlays** — `Spotlight` (command
  palette), `Tabs`, `Stepper`, `Pagination`, `Breadcrumbs` → the
  `plocks-navigation` skill.
- **Tables, lists and other data display** — `Table`, `DataTable`, `Tree`,
  `Timeline`, `Accordion`, `Badge`, `Chip`, `Avatar` → the
  `plocks-data-display` skill.
- **Form inputs and validation** → `plocks-forms`. **Install and
  provider wiring** → `plocks-setup`. **Theme tokens and variant
  colors** → `plocks-theming`. **Screen layout** →
  `plocks-layout`.
