---
name: platform-blocks-data-display
description: Display data with the @platform-blocks/ui React Native library. Use when rendering tabular data with DataTable (sorting, filtering, pagination, selection, bulk actions, server-side/manual pagination) or the low-level Table compound (Table.Thead/Tbody/Tr/Th/Td), building label/value detail lists with DataList, hierarchical Tree views with expansion/checkboxes/lazy loading, event feeds with Timeline, expandable sections with Accordion/Collapse/Spoiler, list rows with ListGroup, or status and identity chrome with Badge, Chip, Avatar/AvatarGroup, Indicator, Rating, Gauge, QRCode and Markdown.
---

# Platform Blocks Data Display

Components for showing data that already exists — tables, lists, trees, feeds,
and the small status chrome around them. All imports are from the package root:

```tsx
import {
  DataTable, Table, DataList, Tree, Timeline, Accordion, Collapse, Spoiler,
  ListGroup, ListGroupItem, ListGroupDivider, ListGroupBody,
  Badge, Chip, Avatar, AvatarGroup, Indicator, Rating, Gauge, QRCode, Markdown,
} from '@platform-blocks/ui';
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
| Data grid with sort / filter / paginate / select | `DataTable` |
| Hand-built table markup, full control | `Table` + `Table.Thead`/`Tbody`/`Tr`/`Th`/`Td` |
| Label → value detail pairs | `DataList` |
| Hierarchy, folders, nav trees | `Tree` |
| Chronological events | `Timeline` |
| Collapsible Q&A / settings sections | `Accordion` |
| One thing that opens and closes | `Collapse` |
| Long text that needs a "show more" | `Spoiler` |
| Simple stacked rows | `ListGroup` |
| Count / status pill | `Badge`, `Chip` |
| Person or entity | `Avatar`, `AvatarGroup` |
| Dot or count on a corner | `Indicator` |
| Stars | `Rating` |
| Single metric dial | `Gauge` (exported, but has no docs page — see pitfall 12) |
| Rendered Markdown | `Markdown` |

## Two API shapes — know which one you are using

Most components here are **data-driven** (`data` / `items` arrays), not
compound. Only three take compound children:

| Component | Shape |
| --- | --- |
| `DataTable`, `Tree`, `Accordion` | **data arrays** — `data`/`items` props |
| `Table` | compound — `Table.Thead`, `Tbody`, `Tr`, `Th`, `Td`, `Tfoot`, `Caption`, `ScrollContainer` (or pass `data` for auto rows) |
| `Timeline` | compound — `Timeline.Item` |
| `DataList` | either — `data` array, or `DataList.Item` / `.ItemLabel` / `.ItemValue` |

There is **no `Accordion.Item`** in the public API (see pitfall 1).

## DataTable

The workhorse. Client-side by default: give it `data` + `columns` and it sorts,
filters, searches and paginates in memory.

```tsx
<DataTable
  data={users}
  columns={[
    { key: 'name',  header: 'Name',  accessor: 'name',  sortable: true },
    { key: 'email', header: 'Email', accessor: 'email', filterable: true },
    {
      key: 'status',
      header: 'Status',
      accessor: (row) => row.status,
      cell: (value) => <Badge color={value === 'active' ? 'success' : 'gray'}>{value}</Badge>,
    },
  ]}
  searchable
  selectable
  getRowId={(row) => row.id}
  onRowClick={(row) => open(row)}
  variant="striped"
  density="normal"
/>
```

`DataTableColumn<T>`: `key`, `header`, `accessor` (a key of `T` **or** a
function), `cell(value, row, index)`, `sortable`, `compare`, `filterable`,
`filterType`, `filterOptions`, `width`/`minWidth`/`maxWidth`.

State props all come in controlled pairs — `sortBy`/`onSortChange`,
`filters`/`onFilterChange`, `searchValue`/`onSearchChange`,
`pagination`/`onPaginationChange`, `selectedRows`/`onSelectionChange`. Omit them
to let the table own that state.

**Server-side tables** set `manualPagination` — then `data` is treated as the
already-fetched page, no client-side slicing/sorting/filtering happens, and
`pagination.total` is required for the page count. Use every control in
controlled mode and refetch in the callbacks.

Also available: `loading`, `error`, `emptyMessage`, `bulkActions`,
`showColumnFilters`, `editMode`/`onCellEdit`, `paginationProps` (forwarded to
the footer `Pagination`), and `id` for persisting user column preferences.

## Table (low level)

```tsx
<Table striped highlightOnHover withTableBorder tabularNums>
  <Table.Thead>
    <Table.Tr><Table.Th>Item</Table.Th><Table.Th>Qty</Table.Th></Table.Tr>
  </Table.Thead>
  <Table.Tbody>
    {rows.map((r) => (
      <Table.Tr key={r.id}><Table.Td>{r.name}</Table.Td><Table.Td>{r.qty}</Table.Td></Table.Tr>
    ))}
  </Table.Tbody>
</Table>
```

Wrap in `Table.ScrollContainer` when columns overflow. `tabularNums` aligns
digits. `variant="vertical"` turns the first column into row headers.

## Tree

Data-driven with `TreeNode<T>` = `{ id, label, children?, hasChildren?, href?,
startOpen?, icon?, disabled?, selectable?, data? }`.

```tsx
<Tree
  data={nodes}
  collapsible
  showGuides
  selectionMode="single"
  checkboxes
  cascadeCheck
  onSelectionChange={(ids, node) => select(node)}
  loadChildren={async (node) => fetchChildren(node.id)}
  renderEndSection={(node) => <Badge>{node.data?.count}</Badge>}
/>
```

Expansion, selection and checking each have a controlled pair
(`expandedIds`/`onExpandedIdsChange`, `selectedIds`/`onSelectionChange`,
`checkedIds`/`onCheckedChange`) and an uncontrolled `default*` variant. For lazy
branches, set `hasChildren: true` on the node so the caret renders before the
children exist — `loadChildren` fires on first expand.

## Timeline

Compound. `active` highlights everything **before** that index.

```tsx
<Timeline active={2} bulletSize={16} lineWidth={2}>
  <Timeline.Item title="Ordered" timestamp="Mar 3">
    <Text>Payment captured.</Text>
  </Timeline.Item>
  <Timeline.Item title="Shipped" timestamp="Mar 5" lineVariant="dashed" />
  <Timeline.Item title="Delivered" timestamp="Mar 7" />
</Timeline>
```

`centerMode` renders one central spine so items can sit on both sides.

## Accordion / Collapse / Spoiler

`Accordion` is **data-driven** — `items: { key, title, content, disabled? }[]`:

```tsx
<Accordion
  items={faqs.map((f) => ({ key: f.id, title: f.question, content: <Text>{f.answer}</Text> }))}
  type="single"                 // 'single' | 'multiple'
  defaultExpanded={[faqs[0].id]}
  variant="default"
  chevronPosition="end"
/>
```

`Collapse` is one show/hide region — note its prop is **`isCollapsed`**
(inverted, and required), not `opened`. `Spoiler` truncates to `maxHeight` and
adds a show-more toggle (`showLabel` / `hideLabel`).

## Status chrome

```tsx
<Badge color="success" variant="light">Active</Badge>
<Badge onRemove={() => remove(tag)}>{tag}</Badge>          {/* removable */}

<Chip variant="surface" dot onPress={toggle}>Filter</Chip>  {/* 'surface' ignores color */}

<Avatar src={user.avatarUrl} fallback={initials} label={user.name} description={user.role} online />
<AvatarGroup>{users.map((u) => <Avatar key={u.id} src={u.avatarUrl} />)}</AvatarGroup>

<Indicator label={9} placement="top-right" color="error">
  <IconButton icon="bell" accessibilityLabel="Notifications" />
</Indicator>
```

`Avatar.fallback` takes an initials string **or** a node. `Indicator.label`
auto-sizes the dot and picks a contrast-aware text color; `children` is for
custom content instead.

## Accessibility

- `DataTable` rows that act as links/buttons need `onRowClick` plus a
  meaningful cell — a clickable row with only an icon is unreachable.
- Give `Avatar` an `accessibilityLabel`; initials are not announced usefully.
- `Badge`/`Chip` used as status must not be the *only* signal — pair color with
  text, since color alone fails WCAG.
- `Tree` supports keyboard focus and selection; keep `label` meaningful because
  `renderLabel` output is what sighted users see, not what is announced.
- `Indicator` count dots need an `accessibilityLabel` on the wrapped control
  ("Notifications, 9 unread") — the dot itself is decorative.

## Pitfalls (verified against source)

1. **There is no public `Accordion.Item`.** The item component is deliberately
   unexported (`AccordionNamespace` is internal and not a root export). Build
   accordions from the `items` array.
2. **`DataTable` `accessor` is not optional** — `key` alone does not read the
   value. Pass a property name or `(row) => value`.
3. **`manualPagination` requires `pagination.total`.** Without it the footer
   cannot compute page count and the table looks stuck on page 1.
4. **`DataTable` and `Tree` virtualization degrade without
   `@shopify/flash-list`.** On v1.0.1+ the dependency is optional: missing it,
   `DataTable virtual` and `Tree virtualized` render every row in a `ScrollView`
   instead of failing — correct but slow on large sets.
5. **`Timeline` `active` is an index, not an id**, and it highlights items
   *before* it. `reverseActive` flips that direction.
6. **`Tree` lazy branches need `hasChildren: true`.** Without it a node with no
   `children` array is a leaf, no caret renders, and `loadChildren` never fires.
7. **`Chip variant="surface"` ignores `color`** — it fills from background
   tokens by design. Use another variant when you need a palette color.
8. **`DataList` ignores `children` when `data` is set.** Pick one shape.
9. **`Accordion` persists expanded state across remounts** (`autoPersist`
   defaults to `true`, keyed by an auto hash). Set `persistKey` for a stable key
   or run it controlled if you need a clean slate each mount.
10. **`Collapse` takes `isCollapsed`, not `opened`** — the boolean is inverted
    relative to every other open/close prop in the library, and it is required.
    `<Collapse isCollapsed={!opened}>`.
11. **`Gauge` has no documentation page.** It is a real root export, but it
    carries no `meta/component.md`, so there is no
    `/llms/components/Gauge.md` to fall back to and no generated prop table in
    `props.md`. Read `packages/ui/src/components/Gauge/types.ts` in the
    monorepo, or use `Ring` — which is documented and covers most single-metric
    dials.
12. **`Table` is layout only** — no sorting, filtering, or pagination. Reach for
    `DataTable` before hand-rolling those on top of `Table`.

## Anything this skill does not cover

This skill covers tables, lists, trees, feeds, and the status chrome around
them. Platform Blocks is much larger — 97 components, 25 charts, and 18 hooks.
Do not guess an API for something outside this scope; fetch the generated docs
instead:

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

- **Navigation** — `Tabs`, `Breadcrumbs`, `Pagination` as a standalone
  control, `Stepper`, `Spotlight`, `TableOfContents`, `Link` → the
  `platform-blocks-navigation` skill.
- **Overlays and feedback** — `Dialog`, `Popover`, `Menu`, `Toast`,
  `Alert`, `Skeleton`, `LoadingOverlay`, `Progress` → the
  `platform-blocks-feedback-overlays` skill.
- **Charts** — every quantitative visualization lives in
  `@platform-blocks/charts` → the `platform-blocks-charts` skill.
  `Gauge` and `Ring` here are single-metric dials, not charts.
- **Editable inputs** → `platform-blocks-forms`. **Install and provider
  wiring** → `platform-blocks-setup`. **Theme tokens** →
  `platform-blocks-theming`. **Page layout** → `platform-blocks-layout`.
