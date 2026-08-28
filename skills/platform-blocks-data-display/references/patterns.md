# Platform Blocks data display — copy-paste patterns

Complete, runnable examples. Verified against `packages/ui/src`.

## Client-side DataTable

Everything in memory — the table owns sort, filter, search, paging and
selection state.

```tsx
import { Badge, Column, DataTable, Text, Title } from '@platform-blocks/ui';

type User = { id: string; name: string; email: string; role: string; status: 'active' | 'invited' };

export function UsersTable({ users }: { users: User[] }) {
  const [selected, setSelected] = useState<(string | number)[]>([]);

  return (
    <Column gap="md">
      <Title order={2}>Team</Title>

      <DataTable<User>
        data={users}
        getRowId={(row) => row.id}
        columns={[
          { key: 'name', header: 'Name', accessor: 'name', sortable: true },
          { key: 'email', header: 'Email', accessor: 'email', filterable: true, filterType: 'text' },
          {
            key: 'role',
            header: 'Role',
            accessor: 'role',
            filterable: true,
            filterType: 'select',
            filterOptions: [
              { label: 'Admin', value: 'admin' },
              { label: 'Member', value: 'member' },
            ],
          },
          {
            key: 'status',
            header: 'Status',
            accessor: (row) => row.status,
            cell: (value: User['status']) => (
              <Badge variant="light" color={value === 'active' ? 'success' : 'gray'}>
                {value}
              </Badge>
            ),
          },
        ]}
        searchable
        searchPlaceholder="Search team…"
        selectable
        selectedRows={selected}
        onSelectionChange={setSelected}
        bulkActions={[
          { key: 'remove', label: 'Remove', action: (ids) => removeUsers(ids as string[]) },
        ]}
        onRowClick={(row) => openProfile(row.id)}
        variant="striped"
        density="normal"
        emptyMessage="No teammates yet."
      />
    </Column>
  );
}
```

## Server-side DataTable (`manualPagination`)

`data` is the current page only. `pagination.total` is required, and every
control must be controlled so you can refetch.

```tsx
import { DataTable } from '@platform-blocks/ui';
import type { DataTableFilter, DataTableSort } from '@platform-blocks/ui';

export function ServerUsersTable() {
  const [page, setPage] = useState({ page: 1, pageSize: 25, total: 0 });
  const [sortBy, setSortBy] = useState<DataTableSort[]>([]);
  const [filters, setFilters] = useState<DataTableFilter[]>([]);
  const [search, setSearch] = useState('');
  const [rows, setRows] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    api
      .listUsers({ page: page.page, pageSize: page.pageSize, sortBy, filters, search })
      .then((res) => {
        if (cancelled) return;
        setRows(res.rows);
        setPage((p) => ({ ...p, total: res.total }));   // total drives the page count
      })
      .finally(() => !cancelled && setLoading(false));
    return () => { cancelled = true; };
  }, [page.page, page.pageSize, sortBy, filters, search]);

  return (
    <DataTable<User>
      data={rows}
      loading={loading}
      manualPagination
      pagination={page}
      onPaginationChange={setPage}
      sortBy={sortBy}
      onSortChange={setSortBy}
      filters={filters}
      onFilterChange={setFilters}
      searchValue={search}
      onSearchChange={setSearch}
      searchable
      getRowId={(row) => row.id}
      columns={columns}
      paginationProps={{ siblings: 1, showSizeChanger: true }}
    />
  );
}
```

## Hand-built Table

When you need exact markup and no table machinery.

```tsx
import { Table, Text } from '@platform-blocks/ui';

<Table.ScrollContainer>
  <Table striped highlightOnHover withTableBorder tabularNums fullWidth>
    <Table.Caption>Q3 revenue by region</Table.Caption>
    <Table.Thead>
      <Table.Tr>
        <Table.Th>Region</Table.Th>
        <Table.Th>Revenue</Table.Th>
        <Table.Th>Change</Table.Th>
      </Table.Tr>
    </Table.Thead>
    <Table.Tbody>
      {regions.map((r) => (
        <Table.Tr key={r.id}>
          <Table.Td>{r.name}</Table.Td>
          <Table.Td>{currency(r.revenue)}</Table.Td>
          <Table.Td>
            <Text colorVariant={r.change >= 0 ? 'success' : 'error'}>
              {r.change >= 0 ? '+' : ''}{r.change}%
            </Text>
          </Table.Td>
        </Table.Tr>
      ))}
    </Table.Tbody>
    <Table.Tfoot>
      <Table.Tr>
        <Table.Td>Total</Table.Td>
        <Table.Td>{currency(total)}</Table.Td>
        <Table.Td />
      </Table.Tr>
    </Table.Tfoot>
  </Table>
</Table.ScrollContainer>
```

## Detail panel with DataList

```tsx
import { Card, DataList, Title } from '@platform-blocks/ui';

<Card p="lg">
  <Title order={3} mb="md">Invoice</Title>
  <DataList
    orientation="horizontal"
    labelWidth={140}
    withDivider
    data={[
      { label: 'Number', value: invoice.number },
      { label: 'Issued', value: formatDate(invoice.issuedAt) },
      { label: 'Due', value: formatDate(invoice.dueAt) },
      { label: 'Status', value: <Badge color="success" variant="light">Paid</Badge> },
      { label: 'Total', value: currency(invoice.total) },
    ]}
  />
</Card>
```

Do not also pass `children` — `data` wins and the children are ignored.

## File tree with lazy loading and checkboxes

`hasChildren: true` is what makes a caret appear before children exist.

```tsx
import { Badge, Tree } from '@platform-blocks/ui';
import type { TreeNode } from '@platform-blocks/ui';

const initial: TreeNode<{ count?: number }>[] = [
  { id: 'src', label: 'src', hasChildren: true, startOpen: true },
  { id: 'docs', label: 'docs', hasChildren: true },
  { id: 'readme', label: 'README.md' },
];

<Tree
  data={initial}
  collapsible
  showGuides
  size="sm"
  selectionMode="single"
  checkboxes
  cascadeCheck
  loadChildren={async (node) => {
    const children = await api.listDirectory(node.id);
    return children.map((c) => ({
      id: c.path,
      label: c.name,
      hasChildren: c.isDirectory,
      data: { count: c.itemCount },
    }));
  }}
  onSelectionChange={(ids, node) => openFile(node)}
  onCheckedChange={(ids) => setStagedPaths(ids)}
  renderEndSection={(node) =>
    node.data?.count ? <Badge variant="subtle">{node.data.count}</Badge> : null
  }
/>
```

## Order timeline

`active` is an index; items *before* it are highlighted.

```tsx
import { Icon, Text, Timeline } from '@platform-blocks/ui';

<Timeline active={currentStep} bulletSize={18} lineWidth={2}>
  <Timeline.Item
    title="Order placed"
    timestamp="Mar 3, 09:12"
    bullet={<Icon name="check" size="xs" />}
  >
    <Text variant="small" colorVariant="secondary">Payment captured.</Text>
  </Timeline.Item>

  <Timeline.Item title="Preparing" timestamp="Mar 4, 14:02">
    <Text variant="small" colorVariant="secondary">Picked and packed.</Text>
  </Timeline.Item>

  <Timeline.Item title="Shipped" timestamp="Mar 5, 08:30" lineVariant="dashed">
    <Text variant="small" colorVariant="secondary">Tracking #{order.tracking}</Text>
  </Timeline.Item>

  <Timeline.Item title="Delivered" colorVariant="success.5" />
</Timeline>
```

## FAQ accordion

Data-driven — there is no `Accordion.Item`.

```tsx
import { Accordion, Text } from '@platform-blocks/ui';

{/* persistKey gives the saved expansion state a stable key across remounts */}
<Accordion
  type="single"
  variant="default"
  density="comfortable"
  persistKey="faq"
  defaultExpanded={[faqs[0].id]}
  items={faqs.map((faq) => ({
    key: faq.id,
    title: faq.question,
    content: <Text>{faq.answer}</Text>,
  }))}
  onExpandedChange={(keys) => track('faq_open', keys)}
/>
```

To start closed on every mount, run it controlled instead:

```tsx
const [expanded, setExpanded] = useState<string[]>([]);
<Accordion items={items} expanded={expanded} onExpandedChange={setExpanded} />
```

## Show/hide a region

`Collapse` takes `isCollapsed` — inverted from every other open/close prop.

```tsx
import { Button, Collapse, useDisclosure } from '@platform-blocks/ui';

const [opened, { toggle }] = useDisclosure(false);

<>
  <Button title={opened ? 'Hide details' : 'Show details'} variant="ghost" onPress={toggle} />
  <Collapse isCollapsed={!opened}>
    <AdvancedSettings />
  </Collapse>
</>
```

## Identity and status row

```tsx
import { Avatar, AvatarGroup, Badge, Chip, Flex, Indicator, IconButton } from '@platform-blocks/ui';

<Flex direction="row" align="center" justify="space-between">
  <Avatar
    src={user.avatarUrl}
    fallback={initials(user.name)}
    label={user.name}
    description={user.role}
    online={user.isOnline}
    accessibilityLabel={`${user.name}, ${user.role}`}
  />

  <Flex direction="row" align="center" gap="sm">
    <Badge variant="light" color={user.plan === 'pro' ? 'primary' : 'gray'}>
      {user.plan}
    </Badge>

    <AvatarGroup>
      {collaborators.slice(0, 4).map((c) => (
        <Avatar key={c.id} src={c.avatarUrl} size="sm" accessibilityLabel={c.name} />
      ))}
    </AvatarGroup>

    <Indicator label={unread} placement="top-right" color="error" invisible={unread === 0}>
      <IconButton
        icon="bell"
        variant="ghost"
        onPress={openInbox}
        accessibilityLabel={`Notifications, ${unread} unread`}
      />
    </Indicator>
  </Flex>
</Flex>
```

## Filter pills

`variant="surface"` is the neutral fill — it ignores `color` by design.

```tsx
<Flex direction="row" wrap="wrap" gap="xs">
  {facets.map((facet) => (
    <Chip
      key={facet.key}
      variant={facet.active ? 'light' : 'surface'}
      color={facet.active ? 'primary' : undefined}
      dot={facet.active}
      onPress={() => toggleFacet(facet.key)}
      onRemove={facet.active ? () => clearFacet(facet.key) : undefined}
    >
      {facet.label}
    </Chip>
  ))}
</Flex>
```
