# plocks data display — API reference

All symbols import from `@plocks/ui`. Verified against
`packages/ui/src` (DataTable, Table, DataList, Tree, Timeline, Accordion,
Collapse, Spoiler, ListGroup, Badge, Chip, Avatar, Indicator, Rating, Gauge,
QRCode, Markdown).

## Root exports

```ts
DataTable
Table                        // Table.Thead .Tbody .Tfoot .Tr .Th .Td .Caption .ScrollContainer
DataList                     // DataList.Item .ItemLabel .ItemValue
Tree
Timeline                     // Timeline.Item
Accordion                    // items-driven; NO public Accordion.Item
Collapse, Spoiler
ListGroup, ListGroupItem, ListGroupDivider, ListGroupBody
Badge, Chip
Avatar, AvatarGroup
Indicator, Rating, Gauge, QRCode, Markdown
```

## DataTable

`DataTableProps<T>`:

| Prop | Type | Notes |
| --- | --- | --- |
| `data` | `T[]` | **required** |
| `columns` | `DataTableColumn<T>[]` | **required** |
| `id` | `string` | stable id for persisting user column preferences |
| `loading` | `boolean` | |
| `error` | `string \| null` | replaces the body when set |
| `emptyMessage` | `string` | |
| `searchable` / `searchPlaceholder` | | global search input |
| `searchValue` / `onSearchChange` | `string` / `(v) => void` | controlled search |
| `sortBy` / `onSortChange` | `DataTableSort[]` | controlled sort |
| `filters` / `onFilterChange` | `DataTableFilter[]` | controlled filters |
| `showColumnFilters` | `boolean` | always-visible filter row under the headers; text input for text/number/date, dropdown for `select`/`boolean` (options derived from data when `filterOptions` is omitted) |
| `pagination` / `onPaginationChange` | `DataTablePagination` | controlled paging |
| `manualPagination` | `boolean` | server-side mode — see below |
| `paginationProps` | `Omit<PaginationProps, 'current' \| 'total' \| 'onChange'>` | forwarded to the footer `Pagination` (`siblings`, `boundaries`, `variant`, `size`, `showTotal`, `showSizeChanger`, …) |
| `selectable` | `boolean` | |
| `selectedRows` / `onSelectionChange` | `(string \| number)[]` | |
| `getRowId` | `(row: T, index: number) => string \| number` | |
| `onRowClick` | `(row: T, index: number) => void` | |
| `editMode` / `onEditModeChange` / `onCellEdit` | | `onCellEdit(rowIndex, columnKey, newValue)` |
| `bulkActions` | `{ key, label, icon?, action(selected, data) }[]` | |
| `variant` | `'default' \| 'striped' \| 'bordered'` | |
| `density` | `'compact' \| 'normal' \| 'comfortable'` | |
| `virtual` | `boolean` | FlashList-powered virtualization |

### DataTableColumn<T>

```ts
interface DataTableColumn<T = any> {
  key: string;                                   // unique column id
  header: React.ReactNode;
  accessor: keyof T | ((row: T) => any);         // REQUIRED — key alone won't read the value
  cell?: (value: any, row: T, index: number) => React.ReactNode;
  sortable?: boolean;
  compare?: (a: any, b: any, rowA: T, rowB: T) => number;
  filterable?: boolean;
  filterType?: FilterType;                       // text | number | date | select | boolean
  filterOptions?: { label: string; value: any }[];
  width?: number | string;
  minWidth?: number;
  maxWidth?: number;
}
```

### manualPagination (server-side)

When `true`, `data` is the already-fetched current page. The table does **no**
client-side slicing, filtering, sorting, or search; `pagination.total` is the
authoritative row count and is **required**. Sort/filter/search controls still
fire their callbacks so you can refetch — run them all controlled.

## Table (compound)

| Prop | Type |
| --- | --- |
| `children` | compound rows, or omit and pass `data` |
| `data` | `TableData` — auto-generates rows |
| `horizontalSpacing` / `verticalSpacing` | `'xs'…'xl' \| number` |
| `striped`, `highlightOnHover` | `boolean` |
| `withTableBorder`, `withColumnBorders`, `withRowBorders` | `boolean` |
| `captionSide` | `'top' \| 'bottom'` |
| `layout` | `'auto' \| 'fixed'` |
| `variant` | `'default' \| 'vertical'` (vertical = first column is row headers) |
| `tabularNums` | `boolean` — aligns digits |
| `fullWidth` | `boolean` |
| `columns` | `{ key?, width?, minWidth?, maxWidth?, flex? }[]` for auto-sizing |

Sub-components: `Table.Thead`, `Table.Tbody`, `Table.Tfoot`, `Table.Tr`,
`Table.Th`, `Table.Td`, `Table.Caption`, `Table.ScrollContainer`.

`Table` has no sorting/filtering/pagination — that is `DataTable`.

## DataList

| Prop | Type | Notes |
| --- | --- | --- |
| `data` | `{ label: ReactNode; value: ReactNode }[]` | wins over `children` |
| `children` | `DataList.Item` composition | ignored when `data` is set |
| `orientation` | `DataListOrientation` | direction of each pair |
| `withDivider` | `boolean` | |
| `size` | `ComponentSizeValue` | font size + spacing |
| `spacing` | `ComponentSizeValue \| number` | vertical gap |
| `labelWidth` | `number \| string` | horizontal orientation only |
| `labelColor` / `valueColor` / `dividerColor` | `string` | |

Compound form: `DataList.Item`, `DataList.ItemLabel`, `DataList.ItemValue`.

## Tree

```ts
interface TreeNode<T = any> {
  id: string;
  label: string;
  children?: TreeNode<T>[];
  hasChildren?: boolean;   // marks a branch BEFORE children exist (lazy loading)
  href?: string;
  startOpen?: boolean;
  icon?: React.ReactNode;
  disabled?: boolean;
  selectable?: boolean;    // overrides the global selectionMode
  data?: T;
}
```

| Prop | Type | Notes |
| --- | --- | --- |
| `data` | `TreeNode<T>[]` | **required** |
| `collapsible`, `expandOnClick`, `accordion`, `expandAll` | `boolean` | `accordion` keeps one branch open per level |
| `expandedIds` / `defaultExpandedIds` / `onExpandedIdsChange` / `onToggle` | | `defaultExpandedIds` overrides each node's `startOpen` |
| `selectionMode` | `'none' \| 'single' \| 'multiple'` | |
| `selectedIds` / `defaultSelectedIds` / `onSelectionChange` | | `(ids, node) => void` |
| `onActiveNodeChange` | `(node \| null, ids) => void` | primary node after a selection change |
| `checkboxes`, `cascadeCheck` | `boolean` | |
| `checkedIds` / `defaultCheckedIds` / `onCheckedChange` | | |
| `loadChildren` | `(node) => Promise<TreeNode<T>[]>` | fires on first expand; node needs `hasChildren` |
| `onNavigate` | `(node) => void` | leaf activation, or any node with `href` |
| `onNodePress` | `(node, { isBranch, event }) => boolean \| void` | return `false` to prevent default |
| `renderLabel` | `(node, depth, isOpen, state) => ReactNode` | |
| `renderEndSection` | `(node, state) => ReactNode` | trailing slot |
| `size`, `indent`, `showGuides`, `rowStyle`, `style` | | |
| `filterQuery` | `string` | highlight / hide unmatched |
| `virtualized` | `boolean` | disables expand/collapse animation |

`TreeNodeState` (given to the render props): `{ selected, checked,
indeterminate, expanded, disabled, focused }`.

## Timeline

| Prop | Type | Notes |
| --- | --- | --- |
| `children` | `Timeline.Item`s | **required** |
| `active` | `number` | index; items **before** it are highlighted |
| `reverseActive` | `boolean` | flips that direction |
| `color` | `ThemeColor` | palette token, `primary.5` shade syntax, or any CSS color |
| `titleColor` / `descriptionColor` / `timestampColor` | `string` | defaults for all items |
| `lineWidth` / `bulletSize` | `number` | |
| `align` | `'left' \| 'right'` | |
| `centerMode` | `boolean` | one central spine, items on both sides |
| `size` | `ComponentSizeValue` | |

`Timeline.Item`: `title`, `children`, `timestamp`, `bullet`, `lineVariant`
(`'solid' \| 'dashed' \| 'dotted'`), `color`, `titleColor`, …

## Accordion

| Prop | Type | Default |
| --- | --- | --- |
| `items` | `{ key: string; title: string; content: ReactNode; disabled?: boolean }[]` | **required** |
| `type` | `'single' \| 'multiple'` | `'single'` |
| `defaultExpanded` | `string[]` | `[]` (single: only the first key is used) |
| `expanded` / `onExpandedChange` | `string[]` / `(keys) => void` | controlled |
| `onItemToggle` | `OnAccordionToggle` | per-item, with metadata |
| `variant` | `AccordionVariant` | `'default'` |
| `size` | `SizeValue` | `'md'` |
| `color` | `ThemeColor` | unset = neutral open state |
| `showChevron` / `chevronPosition` | `boolean` / `'start' \| 'end'` | `true` / `'end'` |
| `density` | `'comfortable' \| 'compact' \| 'spacious'` | `'comfortable'` |
| `persistKey` / `autoPersist` | `string` / `boolean` | `autoPersist` **`true`** — expanded state survives remounts |
| `animated` / `transitionDuration` | | `220` ms; `0` = instant, always 0 under reduced motion |
| `style` / `headerStyle` / `contentStyle` / `headerTextStyle` / `titleProps` | | |

`Accordion.Item` is **not** exported — the item component is internal.

## Collapse / Spoiler

**`Collapse`** — `isCollapsed: boolean` (**required, inverted**), `children`,
`duration` / `transitionDuration`, `timing`
(`'linear' | 'ease' | 'ease-in' | 'ease-out' | 'ease-in-out'`), `easing`,
`onAnimationStart`, `onAnimationEnd`, `style`, `contentStyle`.

**`Spoiler`** — `maxHeight`, `showLabel`, `hideLabel`, plus content.

## Status chrome

**`Badge`** — `children` (required), `size`, `variant`
(`'filled' | 'outline' | 'light' | 'subtle' | 'gradient'`, alias `v`), `c`, `onPress`, `startIcon`, `endIcon`, `onRemove`, `removePosition`
(`'left' | 'right'`), `disabled`, `radius`, `shadow`, `textStyle`, `labelProps`.

**`Chip`** — same shape (with `color` in place of `c`, and no `v`) plus `dot` / `dotColor`, and an extra `'surface'`
variant. **`surface` ignores `color`** — it fills from background tokens (one
step darker than the surface behind it), for input tokens and filter pills.

**`Avatar`** — `src` (URL string or `require()` asset), `fallback` (initials
string or node), `size`, `backgroundColor`, `textColor`, `online`,
`indicatorColor`, `label`, `description`, `gap`, `showText`,
`accessibilityLabel`, `fallbackProps` / `labelProps` / `descriptionProps`.
`AvatarGroup` stacks them.

**`Indicator`** — `label` (count; auto-sizes the dot, contrast-aware text) or
`children` (custom content), `size`, `color`, `borderColor`, `borderWidth`,
`placement` (`'top-left' | 'top-right' | 'bottom-left' | 'bottom-right'`),
`offset`, `invisible`, `labelProps`.

**`Rating`**, **`Gauge`**, **`QRCode`**, **`Markdown`** — see their pages at
`https://plocks.dev/llms/components/<Name>.md`.

## Optional dependency

`@shopify/flash-list` powers `DataTable virtual` and `Tree virtualized`. It is
optional — without it both render every row inside a
`ScrollView` (correct output, no virtualization) and log a dev warning.
