# plocks feedback & overlays — copy-paste patterns

Complete, runnable examples. Verified against `packages/ui/src`.

## App root — providers in the right order

`PlocksProvider` brings the overlay engine; `ToastProvider` and
`DialogProvider` you add yourself.

```tsx
import {
  PlocksProvider,
  ToastProvider,
  DialogProvider,
} from '@plocks/ui';

export default function App() {
  return (
    <PlocksProvider themeModeConfig={{ initialMode: 'auto' }}>
      <ToastProvider defaultPosition="top" limit={3} autoHide={4000}>
        <DialogProvider>
          <RootNavigator />
        </DialogProvider>
      </ToastProvider>
    </PlocksProvider>
  );
}
```

## Save flow — toast on success and failure

```tsx
import { Button, useToast } from '@plocks/ui';

function SaveButton({ draft }: { draft: Draft }) {
  const toast = useToast();
  const [saving, setSaving] = useState(false);

  const save = async () => {
    setSaving(true);
    try {
      await api.save(draft);
      toast.success('Draft saved');
    } catch (error) {
      toast.error({
        title: 'Could not save',
        message: error instanceof Error ? error.message : 'Unknown error',
        autoHide: 0,                       // stay until dismissed
        actions: [{ label: 'Retry', onPress: save }],
      });
    } finally {
      setSaving(false);
    }
  };

  return <Button title="Save" loading={saving} onPress={save} />;
}
```

### The same thing with `toast.promise`

One toast that transitions pending → success/error. Needs `ToastProvider`
mounted — without it this silently does nothing.

```tsx
const toast = useToast();

toast.promise(api.upload(file), {
  pending: { title: 'Uploading…', loading: true, autoHide: 0 },
  success: (result) => ({ title: `Uploaded ${result.name}` }),
  error: (err) => ({ title: 'Upload failed', message: String(err) }),
});
```

### Toasts from outside React

Service modules, API clients and interceptors have no hooks, and the library
exports no module-level toast object — `useToast()` is the only public entry
point. Bridge it yourself: mount a one-line component inside `ToastProvider`
that publishes the API to a module-level handle.

```tsx
// lib/toastBridge.tsx
import { useEffect } from 'react';
import { useToast } from '@plocks/ui';

type ToastApi = ReturnType<typeof useToast>;

let api: ToastApi | null = null;
const queued: ((api: ToastApi) => void)[] = [];

/** Mount once, inside ToastProvider. */
export function ToastBridge() {
  const toast = useToast();

  useEffect(() => {
    api = toast;
    // Replay anything raised before the provider mounted.
    while (queued.length) queued.shift()!(toast);
    return () => {
      if (api === toast) api = null;
    };
  }, [toast]);

  return null;
}

/** Safe to call from anywhere, including before mount. */
export const toast = {
  success: (message: string) => run((a) => a.success(message)),
  error: (message: string) => run((a) => a.error(message)),
  info: (message: string) => run((a) => a.info(message)),
};

function run(fn: (api: ToastApi) => void) {
  if (api) fn(api);
  else queued.push(fn);
}
```

```tsx
// app/_layout.tsx
<ToastProvider>
  <ToastBridge />
  <RootNavigator />
</ToastProvider>
```

```tsx
// api/client.ts — no React involved
import { toast } from '../lib/toastBridge';

http.interceptors.response.use(undefined, (error) => {
  if (error.response?.status === 401) toast.error('Session expired');
  return Promise.reject(error);
});
```

## Confirm dialog — the working version

`useSimpleDialog().confirm()` does not close itself (its buttons call
`closeDialog('')`). Pass your own `id` to `openDialog` and close by that id:

```tsx
import { useCallback } from 'react';
import { Button, Flex, Text, useDialog } from '@plocks/ui';

export function useConfirm() {
  const { openDialog, closeDialog } = useDialog();

  return useCallback(
    (opts: {
      title?: string;
      message: string;
      confirmText?: string;
      cancelText?: string;
      danger?: boolean;
      onConfirm: () => void;
      onCancel?: () => void;
    }) => {
      const id = `confirm-${Date.now()}`;
      const dismiss = (fn?: () => void) => () => {
        closeDialog(id);
        fn?.();
      };

      openDialog({
        id,
        variant: 'modal',
        title: opts.title ?? 'Are you sure?',
        content: (
          <Flex direction="column" gap="lg">
            <Text>{opts.message}</Text>
            <Flex direction="row" gap="sm" justify="flex-end">
              <Button
                title={opts.cancelText ?? 'Cancel'}
                variant="outline"
                onPress={dismiss(opts.onCancel)}
              />
              <Button
                title={opts.confirmText ?? 'Confirm'}
                variant="filled"
                color={opts.danger ? 'error' : undefined}
                onPress={dismiss(opts.onConfirm)}
              />
            </Flex>
          </Flex>
        ),
        onClose: opts.onCancel,
      });

      return id;
    },
    [openDialog, closeDialog],
  );
}

// usage
const confirm = useConfirm();
confirm({
  title: 'Delete project',
  message: 'This permanently removes the project and its data.',
  confirmText: 'Delete',
  danger: true,
  onConfirm: () => remove(project.id),
});
```

## Controlled dialog with a form

`visible` is required and fully controlled — pair it with `useDisclosure`.

```tsx
import {
  Button, Dialog, Flex, Input, Text, useDisclosure, useToast,
} from '@plocks/ui';

function RenameProject({ project }: { project: Project }) {
  const [opened, { open, close }] = useDisclosure(false);
  const [name, setName] = useState(project.name);
  const toast = useToast();

  const submit = async () => {
    await api.rename(project.id, name);
    close();
    toast.success('Renamed');
  };

  return (
    <>
      <Button title="Rename" variant="outline" onPress={open} />

      <Dialog
        opened={opened}
        onClose={close}
        title="Rename project"
        variant="modal"
        autoFocus
        w={420}
      >
        <Flex direction="column" gap="lg">
          <Input
            label="Project name"
            value={name}
            onChangeText={setName}
            onEnter={submit}
          />
          <Flex direction="row" gap="sm" justify="flex-end">
            <Button title="Cancel" variant="outline" onPress={close} />
            <Button title="Save" onPress={submit} disabled={!name.trim()} />
          </Flex>
        </Flex>
      </Dialog>
    </>
  );
}
```

## Bottom sheet

Same component, different `variant`. `radius` rounds the top corners only.

```tsx
<Dialog
  opened={opened}
  onClose={close}
  variant="bottomsheet"
  title="Share"
  radius={20}
  bottomSheetSwipeZone="handle"
>
  <Flex direction="column" gap="sm">
    <Button title="Copy link" variant="ghost" onPress={copyLink} />
    <Button title="Export PDF" variant="ghost" onPress={exportPdf} />
  </Flex>
</Dialog>
```

## Row actions menu

`Menu`'s first child is the trigger; the items are flat sibling exports.

```tsx
import {
  Icon, IconButton, Menu, MenuDivider, MenuDropdown, MenuItem, MenuLabel,
} from '@plocks/ui';

function RowActions({ row }: { row: Row }) {
  return (
    <Menu position="bottom-end" offset={4}>
      <IconButton icon="dots" variant="ghost" accessibilityLabel="Row actions" />

      <MenuDropdown>
        <MenuLabel>Manage</MenuLabel>
        <MenuItem
          startSection={<Icon name="edit" size="sm" />}
          onPress={() => edit(row)}
        >
          Edit
        </MenuItem>
        <MenuItem
          startSection={<Icon name="copy" size="sm" />}
          onPress={() => duplicate(row)}
        >
          Duplicate
        </MenuItem>
        <MenuDivider />
        <MenuItem
          color="danger"
          startSection={<Icon name="trash" size="sm" />}
          onPress={() => remove(row)}
        >
          Delete
        </MenuItem>
      </MenuDropdown>
    </Menu>
  );
}
```

## Popover with interactive content

`Popover` needs an explicit `Popover.Target`. `w="target"` matches the trigger
width.

```tsx
import {
  Button, Checkbox, Flex, Popover, Text, useDisclosure,
} from '@plocks/ui';

function FilterPopover({ filters, onChange }: FilterProps) {
  const [opened, { close, toggle }] = useDisclosure(false);

  return (
    <Popover
      opened={opened}
      onChange={(next) => (next ? toggle() : close())}
      position="bottom-start"
      withArrow
      maw={280}
    >
      <Popover.Target>
        <Button title="Filters" variant="outline" onPress={toggle} />
      </Popover.Target>

      <Popover.Dropdown>
        <Flex direction="column" gap="sm" p="sm">
          <Text fw="semibold">Show</Text>
          {filters.map((filter) => (
            <Checkbox
              key={filter.key}
              label={filter.label}
              checked={filter.enabled}
              onChange={(enabled) => onChange(filter.key, enabled)}
            />
          ))}
        </Flex>
      </Popover.Dropdown>
    </Popover>
  );
}
```

## Tooltip that keyboard users can actually reach

`events.focus` is `false` by default — turn it on, and keep a real
`accessibilityLabel` on the trigger.

```tsx
<Tooltip
  label="Copy the share link"
  position="top"
  withArrow
  openDelay={300}
  events={{ hover: true, focus: true, touch: true }}
>
  <IconButton icon="copy" onPress={copy} accessibilityLabel="Copy share link" />
</Tooltip>
```

## Context menu (right-click / long-press)

`children` is a render prop that hands you the trigger handlers.

```tsx
import { ContextMenu, Card, Text } from '@plocks/ui';
import { Pressable } from 'react-native';

<ContextMenu
  items={[
    { id: 'open', label: 'Open', onSelect: () => open(file) },
    { id: 'rename', label: 'Rename', onSelect: () => rename(file) },
    { id: 'delete', label: 'Delete', onSelect: () => remove(file), danger: true },
  ]}
  longPressDelay={400}
>
  {({ onContextMenu, onPressIn }) => (
    <Pressable onContextMenu={onContextMenu} onPressIn={onPressIn}>
      <Card p="md">
        <Text>{file.name}</Text>
      </Card>
    </Pressable>
  )}
</ContextMenu>
```

## Inline errors with Alert

Use `Alert` for state that belongs in the page, `Toast` for transient events.

```tsx
{error && (
  <Alert severity="error" title="We couldn't load your projects" withCloseButton onClose={dismiss}>
    {error.message}
  </Alert>
)}

{isTrialEnding && (
  <Alert severity="warning" title="Trial ends in 3 days" variant="light">
    Add a payment method to keep your workspaces.
  </Alert>
)}
```

## Loading states

### Skeleton list while data loads

```tsx
import { Card, Column, Flex, Skeleton } from '@plocks/ui';

function ProjectListSkeleton() {
  return (
    <Column gap="sm">
      {Array.from({ length: 5 }).map((_, i) => (
        <Card key={i} p="md">
          <Flex direction="row" align="center" gap="md">
            <Skeleton shape="avatar" size="lg" />
            <Column gap="xs" style={{ flex: 1 }}>
              <Skeleton shape="text" w="60%" />
              <Skeleton shape="text" w="35%" />
            </Column>
          </Flex>
        </Card>
      ))}
    </Column>
  );
}
```

### LoadingOverlay over a section

The wrapper **must** be positioned, or the overlay covers the wrong box.

```tsx
import { Block, LoadingOverlay } from '@plocks/ui';

<Block style={{ position: 'relative' }}>
  <LoadingOverlay visible={isRefreshing} loaderProps={{ variant: 'dots' }} />
  <ProjectTable rows={rows} />
</Block>
```

### Determinate progress

`Progress` takes 0–100, not 0–1.

```tsx
<Column gap="xs">
  <Flex direction="row" justify="space-between">
    <Text variant="small">Uploading</Text>
    <Text variant="small" c="secondary">{percent}%</Text>
  </Flex>
  <Progress value={percent} color="primary" animate striped />
</Column>

{/* circular equivalent */}
<Ring
  value={percent}
  size={120}
  thickness={10}
  caption="Uploaded"
  colorStops={[
    { value: 0, color: '#ef4444' },
    { value: 50, color: '#f59e0b' },
    { value: 100, color: '#22c55e' },
  ]}
/>
```
