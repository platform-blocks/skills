# plocks layout patterns

Complete, copy-paste screen compositions. All imports come from
`@plocks/ui`; the app must be wrapped in `PlocksProvider`
once at the root (see the setup skill for the `SafeAreaProvider` that
`useSafeAreaInsets` reads).

## 1. Basic content screen (ScrollView + Column + Card)

Pattern used by the official expo-template home screen.

```tsx
import { ScrollView } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { Button, Card, Chip, Column, Flex, Text, Title } from '@plocks/ui';

export default function HomeScreen() {
  const insets = useSafeAreaInsets();

  return (
    <ScrollView contentContainerStyle={{ padding: 20, paddingTop: insets.top + 20, gap: 16 }}>
      <Column gap="sm">
        <Title order={1}>Hello, plocks</Title>
        <Text c="secondary">Section subtitle text.</Text>
      </Column>

      <Card variant="elevated" p="lg">
        <Column gap="md">
          <Title order={3}>Card heading</Title>
          {/* Wrapping chip row */}
          <Flex direction="row" gap="xs" wrap="wrap">
            <Chip size="sm" variant="surface">Expo Router</Chip>
            <Chip size="sm" variant="surface">Dark mode</Chip>
            <Chip size="sm" variant="surface">TypeScript</Chip>
          </Flex>
          <Text c="secondary">Body copy inside the card.</Text>
          <Button title="Primary action" variant="filled" onPress={() => {}} />
        </Column>
      </Card>
    </ScrollView>
  );
}
```

## 2. Responsive card grid (12-column, breakpoint spans)

Pattern used on plocks.dev. 1 column on phones, 2 at md (>=640px),
3 at lg (>=960px). `style={{ flex: 1 }}` on the Card makes cards in the same
row equal height; `justify="space-between"` pins the button to the bottom.

```tsx
import { Grid, GridItem, Card, Column, Flex, Text, Button, Chip } from '@plocks/ui';

function TemplateGrid({ templates }: { templates: any[] }) {
  return (
    <Grid columns={12} gap="md">
      {templates.map((t) => (
        <GridItem key={t.key} span={{ base: 12, md: 6, lg: 4 }}>
          <Card variant="elevated" p="md" style={{ flex: 1 }}>
            <Flex direction="column" justify="space-between" gap="md" style={{ flex: 1 }}>
              <Column gap="sm">
                <Flex direction="row" align="center" gap="sm" wrap="wrap">
                  <Text variant="h4" fw="semibold">{t.name}</Text>
                  {!t.available && <Chip size="sm" color="gray" variant="light">coming soon</Chip>}
                </Flex>
                <Text variant="p" c="secondary">{t.description}</Text>
              </Column>
              <Button title="Use template" variant="light" size="sm" onPress={() => {}} />
            </Flex>
          </Card>
        </GridItem>
      ))}
    </Grid>
  );
}
```

Responsive column count instead of responsive spans:

```tsx
<Grid columns={{ base: 1, sm: 2, lg: 4 }} gap="lg">
  {items.map((item) => (
    <GridItem key={item.id} span={1}>
      <Card p="md"><Text>{item.label}</Text></Card>
    </GridItem>
  ))}
</Grid>
```

## 3. Header bar / toolbar row

`Row` shrink-wraps by default — add `fullWidth` so `space-between` has room to work.

```tsx
import { Row, Column, Text, Title, Button } from '@plocks/ui';

function ScreenHeader() {
  return (
    <Row fullWidth align="center" justify="space-between" gap="md">
      <Column gap={0} fullWidth={false}>
        <Title order={2}>Projects</Title>
        <Text variant="small" c="secondary">12 active</Text>
      </Column>
      <Row gap="sm">
        <Button title="Filter" variant="subtle" size="sm" onPress={() => {}} />
        <Button title="New project" variant="filled" size="sm" onPress={() => {}} />
      </Row>
    </Row>
  );
}
```

## 4. List row with fixed leading/trailing and growing middle

No `flex` prop exists on Flex primitives — use `grow` on the middle column.

```tsx
import { Row, Column, Text, Avatar, Icon } from '@plocks/ui';

function ContactRow({ name, subtitle }: { name: string; subtitle: string }) {
  const initials = name.split(' ').map((w) => w[0]).join('');
  return (
    <Row fullWidth align="center" gap="md" py="sm">
      <Avatar fallback={initials} size="md" />
      <Column gap={0} grow={1} shrink={1} fullWidth={false}>
        <Text fw="semibold">{name}</Text>
        <Text variant="small" c="secondary" numberOfLines={1}>{subtitle}</Text>
      </Column>
      <Icon name="chevronRight" size="sm" />
    </Row>
  );
}
```

## 5. Breakpoint-conditional layout (split pane vs stacked)

```tsx
import { Row, Column, Card, useBreakpoint } from '@plocks/ui';

function DetailScreen({ list, detail }: { list: React.ReactNode; detail: React.ReactNode }) {
  const breakpoint = useBreakpoint();               // 'xs' | 'sm' | 'md' | 'lg' | 'xl'
  const isMobile = breakpoint === 'xs' || breakpoint === 'sm';

  if (isMobile) {
    return <Column gap="md">{list}{detail}</Column>;
  }
  return (
    <Row fullWidth gap="lg" align="stretch">
      <Card variant="subtle" p="md" style={{ flex: 1, maxWidth: 360 }}>{list}</Card>
      <Card variant="filled" p="lg" style={{ flex: 2 }}>{detail}</Card>
    </Row>
  );
}
```

## 6. Card with full-bleed image and banded sections

`Card.Section` must be a direct child; `clip` keeps the image inside the radius.

```tsx
import { Card, Column, Row, Text, Title, Button, Image } from '@plocks/ui';

function ArticleCard() {
  return (
    <Card variant="elevated" clip>
      <Card.Section>
        <Image source={{ uri: 'https://example.com/cover.jpg' }} style={{ width: '100%', height: 160 }} />
      </Card.Section>

      <Column gap="sm" pt="md">
        <Title order={4}>Article title</Title>
        <Text variant="small" c="secondary">Teaser copy for the article.</Text>
      </Column>

      <Card.Section withBorder inheritPadding py="sm">
        <Row fullWidth align="center" justify="space-between">
          <Text variant="small" c="secondary">5 min read</Text>
          <Button title="Read" variant="subtle" size="sm" onPress={() => {}} />
        </Row>
      </Card.Section>
    </Card>
  );
}
```

## 7. Nested elevation with Surface

`raised` derives the level from the enclosing Surface — no hard-coded numbers.

```tsx
import { Surface, Column, Text, Title } from '@plocks/ui';

function StatsPanel() {
  return (
    <Surface level={1} padding="lg" radius="lg">
      <Column gap="md">
        <Title order={4}>Overview</Title>
        <Surface raised padding="md" radius="md">
          <Text>This inner panel automatically sits one elevation step above.</Text>
        </Surface>
      </Column>
    </Surface>
  );
}
```

## 8. Centered auth / empty-state screen

```tsx
import { Flex, Column, Card, Title, Text, Button, Input, PasswordInput } from '@plocks/ui';

function SignInScreen() {
  return (
    <Flex direction="column" align="center" justify="center" grow={1} p="lg">
      <Card variant="elevated" p="xl" radius="lg" style={{ width: '100%', maxWidth: 420 }}>
        <Column gap="md">
          <Column gap="xs">
            <Title order={2}>Welcome back</Title>
            <Text c="secondary">Sign in to continue.</Text>
          </Column>
          <Input label="Email" placeholder="you@example.com" fullWidth />
          <PasswordInput label="Password" fullWidth />
          <Button title="Sign in" variant="filled" fullWidth onPress={() => {}} />
        </Column>
      </Card>
    </Flex>
  );
}
```

## 9. AppShell — composed shell with responsive navbar

```tsx
import {
  AppShell, Row, Column, Text, Title, Button, useAppShell,
} from '@plocks/ui';

function Shell({ children }: { children: React.ReactNode }) {
  return (
    <AppShell
      header={{ height: { base: 56, md: 64 } }}
      navbar={{
        width: { base: '100%', md: 240, lg: 280 },
        breakpoint: 'md',                 // below md the navbar becomes a drawer
        collapsedWidth: 72,
        expandOnHover: true,
        startCollapsedDesktop: true,
      }}
      bottomNav={{ height: 64, showOnlyMobile: true }}
      maxContentWidth={1200}
      centerContent
    >
      <AppShell.Header withBorder>
        <HeaderBar />
      </AppShell.Header>

      <AppShell.Navbar withBorder>
        <AppShell.Section grow withScrollArea>
          <NavLinks />
        </AppShell.Section>
        <AppShell.Section>
          <Text variant="small" c="secondary">Current version</Text>
        </AppShell.Section>
      </AppShell.Navbar>

      <AppShell.Main>{children}</AppShell.Main>

      <AppShell.BottomNav
        items={[
          { key: 'home', label: 'Home', icon: <HomeIcon /> },
          { key: 'search', label: 'Search', icon: <SearchIcon /> },
        ]}
        activeKey="home"
        onItemPress={(key) => {/* navigate */}}
      />
    </AppShell>
  );
}

function HeaderBar() {
  const { toggleNavbar, isMobile } = useAppShell();
  return (
    <Row fullWidth align="center" justify="space-between" px="md" gap="md">
      <Row align="center" gap="sm">
        {isMobile && <Button title="Menu" variant="subtle" size="sm" onPress={toggleNavbar} />}
        <Title order={4}>My App</Title>
      </Row>
      <Button title="Sign out" variant="subtle" size="sm" onPress={() => {}} />
    </Row>
  );
}
```

## 10. AppShell — declarative blueprint (defineAppLayout)

Pattern used by plocks.dev (Expo Router). The blueprint lives in a
config module; the root layout mounts it once.

```tsx
// config/appLayout.tsx
import { defineAppLayout, type AppLayoutRuntimeContext } from '@plocks/ui';
import { AppHeader } from '../components/layout/Header';
import { AppNavigation } from '../components/layout/Navigation';
import { MobileBottomBar } from '../components/layout/BottomBar';

export const appLayout = defineAppLayout({
  id: 'app-shell',
  breakpoints: {
    headerHeight: { base: 56, md: 60 },
    navbarWidth: { base: 220, lg: 260 },
    bottomNavHeight: 72,
    padding: { base: 0 },
  },
  layout: { withSafeArea: true, padding: { base: 0 } },
  main: { id: 'main-content', role: 'main', maxWidth: 1800, centerContent: true },
  header: {
    render: (ctx: AppLayoutRuntimeContext) =>
      ctx.isMobile ? <AppHeader compact /> : <AppHeader />,
  },
  navbar: {
    component: AppNavigation,
    show: (ctx) => !ctx.isMobile,
    props: (ctx) => ({ onNavigate: (route: string) => ctx.navigation?.push?.(route) }),
    width: { base: '100%', md: 220, lg: 260 },
    collapsedWidth: 60,
    expandOnHover: true,
    expandOnHoverPush: true,
    autoExpandBreakpoint: 'xl',
    startCollapsedDesktop: true,
  },
  bottomNav: {
    component: MobileBottomBar,
    show: (ctx) => ctx.platform !== 'web' && !ctx.isLandscape,
  },
});
```

```tsx
// app/_layout.tsx (Expo Router)
import React from 'react';
import { Stack, useGlobalSearchParams, usePathname, router } from 'expo-router';
import { AppLayoutProvider, AppLayoutRenderer } from '@plocks/ui';
import { AppProviders } from '../components/layout/Providers'; // wraps PlocksProvider
import { appLayout } from '../config/appLayout';

export default function RootLayout() {
  const params = useGlobalSearchParams();
  const pathname = usePathname();

  const query = React.useMemo(
    () => Object.fromEntries(Object.entries(params ?? {})), [params]);
  const navigation = React.useMemo(() => ({
    push: (path: string) => router.push(path),
    replace: (path: string) => router.replace(path),
    goBack: () => router.back(),
  }), []);

  return (
    <AppProviders>
      <AppLayoutProvider blueprint={appLayout} value={{ query, pathname, navigation }}>
        <AppLayoutRenderer>
          <Stack screenOptions={{ headerShown: false }} />
        </AppLayoutRenderer>
      </AppLayoutProvider>
    </AppProviders>
  );
}
```

## 11. Dashboard stat tiles (Grid + Card + Row)

```tsx
import { Grid, GridItem, Card, Column, Row, Text, Title } from '@plocks/ui';

const STATS = [
  { label: 'Revenue', value: '$12.4k', delta: '+8%' },
  { label: 'Users', value: '1,203', delta: '+3%' },
  { label: 'Errors', value: '17', delta: '-12%' },
  { label: 'Uptime', value: '99.98%', delta: '' },
];

function StatTiles() {
  return (
    <Grid columns={12} gap="md">
      {STATS.map((s) => (
        <GridItem key={s.label} span={{ base: 6, md: 3 }}>
          <Card variant="subtle" p="md" style={{ flex: 1 }}>
            <Column gap="xs">
              <Text variant="small" c="secondary">{s.label}</Text>
              <Row align="baseline" gap="xs">
                <Title order={3}>{s.value}</Title>
                {s.delta ? <Text variant="small" c="secondary">{s.delta}</Text> : null}
              </Row>
            </Column>
          </Card>
        </GridItem>
      ))}
    </Grid>
  );
}
```

## Quick reminders

- Fill remaining space: `grow={1}` (Flex/Row/Column), `fluid` (Block), or `style={{ flex: 1 }}`.
- Equal-height cards in a Grid row: `style={{ flex: 1 }}` on the Card (GridItem's wrapper already stretches).
- `Grid` needs an explicit `gap` (default is 0); `Flex`/`Row`/`Column` already have `gap="sm"`.
- Grid responsive keys: `{ base, sm, md, lg, xl }` — no `xs`.
- Spacing accepts tokens (`p="lg"`) or numbers (`p={20}`); `gap` likewise.
