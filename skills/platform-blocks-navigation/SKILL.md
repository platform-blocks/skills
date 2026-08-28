---
name: platform-blocks-navigation
description: Build navigation with the @platform-blocks/ui React Native library. Use when adding in-page tabs (Tabs with items/activeTab/onTabChange and line/chip/card/folder variants), breadcrumb trails, Pagination controls, multi-step Stepper flows, a Spotlight command palette (spotlight store, cmd+K shortcut), a TableOfContents driven by useScrollSpy, Link and MenuItemButton, or wiring any of these to Expo Router (useRouter/usePathname, SEO-safe anchors, theme bridging) — including how Platform Blocks Tabs differs from expo-router Tabs.
---

# Platform Blocks Navigation

Moving between views, and showing where you are. All imports are from the
package root:

```tsx
import {
  Tabs, Breadcrumbs, Pagination, Stepper, TableOfContents,
  Spotlight, SpotlightProvider, spotlight, useSpotlightStoreInstance,
  Link, MenuItemButton,
  useScrollSpy, useHotkeys, useGlobalHotkeys, useSpotlightToggle,
} from '@platform-blocks/ui';
```

The `Navigation` module (`NavigationContainer`, `createStackNavigator`,
`createDrawerNavigator`, `Screen`, `useNavigation`, `useRoute`) is **subpath
only**: `import { NavigationContainer } from '@platform-blocks/ui/Navigation'`.

Four reference files sit alongside this one:

- `references/api.md` — the curated API: how the pieces compose, the union
  types, the defaults that bite. **Read this first.**
- `references/props.md` — generated, exhaustive prop tables for every component
  in this skill. Look here for the complete surface of a single prop.
- `references/icons.md` — generated, every `name` the built-in `Icon` registry
  accepts. Check it before writing `<Icon name="…">`; unlisted names render
  nothing.
- `references/patterns.md` — complete copy-paste screens.

## The most important distinction: two different `Tabs`

| | `Tabs` from `@platform-blocks/ui` | `Tabs` from `expo-router` |
| --- | --- | --- |
| What it is | In-page tab strip **that renders its own content panels** | Route-based bottom tab navigator |
| Driven by | `items: TabItem[]` with a `content` node per tab | Files under `app/(tabs)/` + `<Tabs.Screen>` |
| URL changes | No | Yes |
| Use for | Sections inside one screen | Top-level app navigation |

The official `expo-template` uses **expo-router's** `Tabs` in
`app/(tabs)/_layout.tsx`. If you are building bottom tab bars, that is the one
you want. Importing the wrong `Tabs` is the single most common mistake here.

## Tabs (in-page)

Data-driven, and it owns the content area — each item carries its own `content`.

```tsx
<Tabs
  items={[
    { key: 'overview', label: 'Overview', content: <Overview /> },
    { key: 'activity', label: 'Activity', content: <Activity />, icon: <Icon name="clock" size="sm" /> },
    { key: 'billing',  label: 'Billing',  content: <Billing />, disabled: !isAdmin },
  ]}
  variant="line"          // 'line' | 'chip' | 'card' | 'folder'
  onTabChange={(key) => track('tab', key)}
/>
```

Uncontrolled by default (first item active). Pass `activeTab` +`onTabChange` to
control it. `orientation="vertical"`, `scrollable` for overflow, `location`
(`'start' | 'end'`) for where the strip sits.

Set **`navigationOnly`** when the tabs should only switch routes and you render
the body yourself — `content` is then ignored.

## Breadcrumbs

```tsx
<Breadcrumbs
  items={[
    { label: 'Home', onPress: () => router.push('/') },
    { label: 'Projects', href: '/projects' },
    { label: project.name },              // last item, no handler = current page
  ]}
  separator="›"
  maxItems={4}                            // collapses the middle
/>
```

## Pagination

Standalone control — `current` is **1-indexed**, `total` is the **page count**,
not the row count.

```tsx
<Pagination
  current={page}
  total={Math.ceil(rowCount / pageSize)}
  onChange={setPage}
  siblings={1}
  boundaries={1}
  showTotal
  totalItems={rowCount}
  hideOnSinglePage
/>
```

`DataTable` renders one of these in its footer already — configure it there via
`paginationProps` instead of adding a second.

## Stepper

Compound: `Stepper.Step` and `Stepper.Completed`. `active` is a **0-based
index**.

```tsx
<Stepper active={step} onStepClick={setStep} orientation="horizontal">
  <Stepper.Step label="Account" description="Email and password"><AccountForm /></Stepper.Step>
  <Stepper.Step label="Profile" description="Tell us about you"><ProfileForm /></Stepper.Step>
  <Stepper.Step label="Confirm"><Review /></Stepper.Step>
  <Stepper.Completed><Done /></Stepper.Completed>
</Stepper>
```

Steps before `active` render as completed. `allowNextStepsSelect` decides
whether users can jump ahead.

## Spotlight (command palette)

`actions` are `SpotlightActionData`: `{ id, label, description?, keywords?,
icon?, onPress? }`. Open it from anywhere with the module-level `spotlight`
object, or bind a shortcut.

```tsx
<Spotlight
  actions={actions}
  shortcut={['mod+k']}
  nothingFound="No matches"
  limit={8}
  highlightQuery
/>

{/* shortcut defaults to ['cmd+k', 'ctrl+k']; 'mod' maps to cmd on Mac, ctrl elsewhere */}

// from anywhere, no hook required
import { spotlight } from '@platform-blocks/ui';
spotlight.open();
```

`variant` is `'modal' | 'bottomsheet' | 'fullscreen'`. `keywords` widen search
matching without showing extra text.

## TableOfContents

**Web only** — it drives off `useScrollSpy`, which needs
`IntersectionObserver` and real DOM headings. It finds headings inside
`container` (a CSS selector) and highlights the one in view.

```tsx
<TableOfContents
  variant="ghost"
  size="sm"
  container="article"
  depthOffset={20}
  minDepthToOffset={1}
  onActiveChange={(id) => setActiveHeading(id)}
/>
```

Pass `initialData` so prerendered pages show the list before hydration.

## Link

```tsx
<Link href="https://platform-blocks.com" external target="_blank">Docs</Link>
<Link onPress={() => router.push('/settings')} variant="hover-underline">Settings</Link>
```

`variant`: `'default' | 'subtle' | 'hover-underline'`. `color` also accepts
`'inherit'` to follow surrounding text.

`Link` does **not** know about your router — `href` is for real URLs. For
in-app routes either pass `onPress`, or use the SEO-safe anchor pattern in
`references/patterns.md`, which emits a real `<a href>` on web (so crawlers,
middle-click and "copy link address" work) while handing plain left-clicks to
the router.

## Keyboard

```tsx
useHotkeys([['mod+k', () => spotlight.open()], ['/', focusSearch]], [focusSearch]);
useGlobalHotkeys('save', ['mod+s', save]);   // app-wide, survives unmount ordering
```

`useScrollSpy(options?, initialData?)` returns `{ items, activeId, … }` if you
want the scroll-spy data without `TableOfContents`.

## Accessibility

- The current page in `Breadcrumbs` should have no `href`/`onPress` — that is
  what marks it as current.
- `Pagination` buttons are labelled via `labels`; override them when localizing.
- `Tabs` manages roving focus; keep `label` a string where you can, since a
  custom node is what screen readers read.
- `Spotlight` needs a visible affordance too — a keyboard shortcut alone is
  invisible to touch users. Pair it with a search button.
- `TableOfContents` is supplementary navigation; never make it the only route to
  a section.

## Pitfalls (verified against source)

1. **Two `Tabs` exist.** `@platform-blocks/ui` `Tabs` is an in-page strip that
   renders `content` panels and does not touch the URL; `expo-router` `Tabs` is
   the bottom tab navigator. Check your import before debugging "my tabs don't
   navigate".
2. **`TabItem.content` is required** unless `navigationOnly` is set — `Tabs`
   renders the body, it is not just a strip.
3. **`Pagination.total` is the number of pages, not rows.** Pass
   `Math.ceil(rowCount / pageSize)`. `totalItems` is the row count, and only
   feeds the `showTotal` summary.
4. **`Pagination.current` is 1-indexed; `Stepper.active` is 0-indexed.** They
   disagree, and both are commonly wired to the same kind of state.
5. **`TableOfContents` and `useScrollSpy` are web-only.** They rely on
   `IntersectionObserver` and DOM heading elements; on native they find nothing
   and render empty.
6. **`Navigation` is subpath-only** — `import { NavigationContainer } from
   '@platform-blocks/ui/Navigation'`. It is not on the package root, and in an
   Expo Router app you almost certainly want Expo Router instead.
7. **`Link` is not a router link.** `href` produces a real link on web and does
   nothing for in-app routes on native. Use `onPress` with `router.push`, or the
   anchor pattern in patterns.md.
8. **`Tabs` persists its active tab** when `persistKey`/`autoPersist` are set —
   a "reset to first tab" expectation will fail until you run it controlled.
9. **`Spotlight` needs its store mounted.** Use `SpotlightProvider` (or
   `useSpotlightStoreInstance` for a local store); the module-level `spotlight`
   helper toggles the shared store and does nothing if nothing is listening.
10. **`Breadcrumbs.maxItems` collapses the middle, not the tail** — the first
    and last entries always survive.

## Anything this skill does not cover

This skill covers in-page navigation controls and how they meet Expo Router.
Platform Blocks is much larger — 97 components, 25 charts, and 18 hooks. Do not
guess an API for something outside this scope; fetch the generated docs instead:

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
'@platform-blocks/ui'`). The two exceptions are `FormLayout` and `AudioPlayer`,
which are subpath-only (`@platform-blocks/ui/FormLayout`, `@platform-
blocks/ui/AudioPlayer`); a few utilities also live on subpaths (e.g.
`validationRules` on `@platform-blocks/ui/Input`). A docs page existing does not
guarantee a root export — `HoverCard`, for instance, is internal and has no page
and no export.

Notably outside this skill:

- **App chrome** — `AppShell` (header, navbar, aside, footer, bottom nav) and
  layout blueprints → the `platform-blocks-layout` skill.
- **Dropdown menus** — `Menu`/`MenuDropdown`/`MenuItem`, `ContextMenu`,
  `Popover`, `Dialog` → the `platform-blocks-feedback-overlays` skill.
  (`Spotlight` is here because it is a navigation surface.)
- **Tables and lists** — `DataTable` and its built-in footer pagination,
  `Tree` as a nav tree, `Timeline` → the `platform-blocks-data-display`
  skill.
- **Form inputs** → `platform-blocks-forms`. **Install and provider wiring**
  → `platform-blocks-setup`. **Theme tokens** → `platform-blocks-theming`.
