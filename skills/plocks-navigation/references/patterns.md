# plocks navigation — copy-paste patterns

Complete, runnable examples. Verified against `packages/ui/src`, the official
`expo-template`, and `apps/plocks.dev`.

## In-page tabs

`Tabs` renders the content panels itself — each item carries its own `content`.

```tsx
import { Icon, Tabs } from '@plocks/ui';

export function ProjectScreen({ project, isAdmin }: Props) {
  const [tab, setTab] = useState('overview');

  return (
    <Tabs
      value={tab}
      onChange={setTab}
      variant="line"
      scrollable
      items={[
        { key: 'overview', label: 'Overview', content: <Overview project={project} /> },
        {
          key: 'activity',
          label: 'Activity',
          subLabel: `${project.eventCount} events`,
          icon: <Icon name="clock" size="sm" />,
          content: <Activity projectId={project.id} />,
        },
        {
          key: 'billing',
          label: 'Billing',
          disabled: !isAdmin,
          content: <Billing project={project} />,
        },
      ]}
      onDisabledTabPress={() => toast.info('Only admins can view billing')}
    />
  );
}
```

### Route-backed tabs

When the tab should change the URL, set `navigationOnly` and render the body
from the router.

```tsx
import { usePathname, useRouter } from 'expo-router';
import { Tabs } from '@plocks/ui';

const TABS = [
  { key: '/settings/profile', label: 'Profile' },
  { key: '/settings/team', label: 'Team' },
  { key: '/settings/billing', label: 'Billing' },
];

export function SettingsTabs() {
  const router = useRouter();
  const pathname = usePathname();

  return (
    <Tabs
      navigationOnly
      value={pathname}
      onChange={(key) => router.push(key as never)}
      items={TABS.map((t) => ({ ...t, content: null }))}
    />
  );
}
```

## Bottom tabs are Expo Router's `Tabs`, not this library's

This is `app/(tabs)/_layout.tsx` from the official template. Note the import.

```tsx
import { Tabs } from 'expo-router';          // <- NOT @plocks/ui
import { useTheme } from '@plocks/ui';

export default function TabsLayout() {
  const theme = useTheme();

  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarActiveTintColor: theme.colors.primary[6],
        tabBarInactiveTintColor: theme.text.secondary,
        tabBarStyle: { backgroundColor: theme.backgrounds.surface },
      }}
    >
      <Tabs.Screen name="index" options={{ title: 'Home' }} />
      <Tabs.Screen name="settings" options={{ title: 'Settings' }} />
    </Tabs>
  );
}
```

## Breadcrumbs wired to the router

The current page gets no handler — that is what marks it as current.

```tsx
import { useRouter } from 'expo-router';
import { Breadcrumbs } from '@plocks/ui';

export function ProjectBreadcrumbs({ project }: { project: Project }) {
  const router = useRouter();

  return (
    <Breadcrumbs
      separator="›"
      maxItems={4}
      items={[
        { label: 'Home', onPress: () => router.push('/') },
        { label: 'Projects', onPress: () => router.push('/projects') },
        { label: project.workspace, onPress: () => router.push(`/projects?ws=${project.workspaceId}`) },
        { label: project.name },
      ]}
    />
  );
}
```

## Standalone pagination

`total` is the **page count**; `totalItems` is the row count and only feeds the
summary line.

```tsx
import { Column, Pagination } from '@plocks/ui';

const pageSize = 20;
const [page, setPage] = useState(1);
const pageRows = rows.slice((page - 1) * pageSize, page * pageSize);

<Column gap="md">
  <ResultList rows={pageRows} />
  <Pagination
    value={page}
    total={Math.ceil(rows.length / pageSize)}
    onChange={setPage}
    totalItems={rows.length}
    showTotal={(total, [from, to]) => `${from}–${to} of ${total}`}
    hideOnSinglePage
    siblings={1}
    boundaries={1}
  />
</Column>
```

Inside a `DataTable`, do not add a second one — configure the built-in footer
control through `paginationProps`.

## Multi-step wizard

`active` is 0-indexed. `Stepper.Completed` renders once you move past the last
step.

```tsx
import { Button, Flex, Stepper } from '@plocks/ui';

export function Onboarding() {
  const [step, setStep] = useState(0);
  const last = 2;

  return (
    <>
      <Stepper active={step} onStepClick={setStep} allowNextStepsSelect={false}>
        <Stepper.Step label="Account" description="Email and password">
          <AccountForm onValid={() => setStep(1)} />
        </Stepper.Step>

        <Stepper.Step label="Profile" description="Tell us about you">
          <ProfileForm />
        </Stepper.Step>

        <Stepper.Step label="Confirm">
          <Review />
        </Stepper.Step>

        <Stepper.Completed>
          <Done />
        </Stepper.Completed>
      </Stepper>

      <Flex direction="row" gap="sm" justify="flex-end" mt="lg">
        <Button
          title="Back"
          variant="outline"
          disabled={step === 0}
          onPress={() => setStep((s) => s - 1)}
        />
        <Button
          title={step === last ? 'Finish' : 'Next'}
          onPress={() => setStep((s) => s + 1)}
        />
      </Flex>
    </>
  );
}
```

## Command palette

Mount the store once, then open it from anywhere.

```tsx
// app/_layout.tsx
import { SpotlightProvider } from '@plocks/spotlight';

<PlocksProvider>
  <SpotlightProvider>
    <AppShellRoutes />
  </SpotlightProvider>
</PlocksProvider>
```

```tsx
// components/CommandPalette.tsx
import { useMemo } from 'react';
import { useRouter } from 'expo-router';
import { Spotlight } from '@plocks/spotlight';

export function CommandPalette({ projects }: { projects: Project[] }) {
  const router = useRouter();

  const actions = useMemo(
    () => [
      {
        id: 'new-project',
        label: 'Create project',
        description: 'Start a new workspace',
        icon: 'plus',
        keywords: ['add', 'new', 'create'],
        onPress: () => router.push('/projects/new'),
      },
      ...projects.map((p) => ({
        id: `project-${p.id}`,
        label: p.name,
        description: 'Open project',
        icon: 'folder',
        keywords: [p.workspace],
        onPress: () => router.push(`/projects/${p.id}`),
      })),
    ],
    [projects, router],
  );

  return (
    <Spotlight
      actions={actions}
      nothingFound="No matches"
      limit={8}
      highlightQuery
      variant="modal"
    />
  );
}
```

```tsx
// anywhere — a visible affordance, because a shortcut alone is invisible on touch
import { Button } from '@plocks/ui';
import { spotlight } from '@plocks/spotlight';

<Button title="Search  ⌘K" variant="outline" onPress={() => spotlight.open()} />
```

`keywords` widen matching without adding visible text. The default shortcut is
already `['cmd+k', 'ctrl+k']`.

## SEO-safe in-app links (web + native)

`Link`'s `href` is for real URLs, and bare `router.push()` leaves no anchor
behind — crawlers see no outgoing links, and readers lose middle-click,
cmd-click and "copy link address". This is the pattern
`apps/plocks.dev` ships: a real `<a href>` on web that hands plain
left-clicks to the router, and a `Pressable` on native.

```tsx
import React, { useCallback } from 'react';
import { Platform, Pressable, StyleSheet, type StyleProp, type ViewStyle } from 'react-native';
import { useRouter } from 'expo-router';

export interface RouteLinkProps {
  href: string;
  children: React.ReactNode;
  style?: StyleProp<ViewStyle>;
  accessibilityLabel?: string;
}

const WEB_BASE_STYLE = {
  display: 'flex' as const,
  flexDirection: 'column' as const,
  color: 'inherit',
  textDecoration: 'none' as const,
  cursor: 'pointer' as const,
};

export const RouteLink: React.FC<RouteLinkProps> = ({
  href,
  children,
  style,
  accessibilityLabel,
}) => {
  const router = useRouter();
  const navigate = useCallback(() => router.push(href as never), [href, router]);

  const handleClick = useCallback(
    (event: any) => {
      // Defer to the browser when the user explicitly asked for it: modified
      // clicks open a new tab, and non-primary buttons aren't ours to intercept.
      if (event.defaultPrevented) return;
      if (event.button !== 0) return;
      if (event.metaKey || event.ctrlKey || event.shiftKey || event.altKey) return;
      event.preventDefault();
      navigate();
    },
    [navigate],
  );

  if (Platform.OS === 'web') {
    return React.createElement(
      'a',
      {
        href,
        onClick: handleClick,
        'aria-label': accessibilityLabel,
        style: { ...WEB_BASE_STYLE, ...(StyleSheet.flatten(style) as object) },
      },
      children,
    );
  }

  return (
    <Pressable
      onPress={navigate}
      style={style}
      accessibilityRole="link"
      accessibilityLabel={accessibilityLabel}
    >
      {children}
    </Pressable>
  );
};
```

On web the wrapper style lands on the anchor itself, which bypasses
react-native-web's style pipeline — so use plain CSS-compatible properties
there (`paddingHorizontal` and friends will not apply).

Plain external links stay simple:

```tsx
<Link href="https://plocks.dev" external target="_blank">Documentation</Link>
```

## Table of contents (web)

Driven by `useScrollSpy` over real DOM headings, so it is web-only. Pass
`initialData` so prerendered pages render the list before hydration.

```tsx
import { Flex, TableOfContents } from '@plocks/ui';

<Flex direction="row" gap="xl" align="flex-start">
  <article style={{ flex: 1 }}>
    <DocBody />
  </article>

  <TableOfContents
    variant="ghost"
    size="sm"
    container="article"
    depthOffset={20}
    initialData={headings}
    onActiveChange={(id) => setActiveHeading(id)}
    style={{ width: 220, position: 'sticky', top: 96 }}
  />
</Flex>
```

Need the data without the UI? `useScrollSpy()` returns the same items plus
`activeId`.

## App-wide keyboard shortcuts

```tsx
import { useGlobalHotkeys, useHotkeys } from '@plocks/ui';
import { spotlight } from '@plocks/spotlight';

// scoped to the mounted component
useHotkeys(
  [
    ['mod+k', () => spotlight.open()],
    ['mod+s', save],
    ['escape', clearSelection],
  ],
  [save, clearSelection],
);

// registered globally by id, independent of mount order
useGlobalHotkeys('save-document', ['mod+s', save]);
```

`mod` is ⌘ on Mac and Ctrl elsewhere. Hotkey strings are lowercased before
parsing, so case does not matter.
