# Setup patterns

## Minimal app root

```tsx
import { Button, Card, PlocksProvider, Text } from '@plocks/ui';

export default function App() {
  return (
    <PlocksProvider>
      <Card p="lg">
        <Text>Welcome to plocks</Text>
        <Button title="Continue" onPress={() => {}} />
      </Card>
    </PlocksProvider>
  );
}
```

## Expo Router root

Put `PlocksProvider` around the router's stack or tabs. When using `themeModeConfig`, call `useTheme()` below the provider and pass a matching theme to React Navigation. The maintained implementation is in [expo-template/app/_layout.tsx](https://github.com/platform-blocks/expo-template/blob/HEAD/app/_layout.tsx). For static web output, copy its [`+html.tsx`](https://github.com/platform-blocks/expo-template/blob/HEAD/app/+html.tsx) script so the stored scheme is applied before hydration.

## Component test

```tsx
import renderer from 'react-test-renderer';
import { PlocksProvider, Text } from '@plocks/ui';

test('renders app content', () => {
  const tree = renderer.create(
    <PlocksProvider><Text>Hello plocks</Text></PlocksProvider>
  );
  expect(JSON.stringify(tree.toJSON())).toContain('Hello plocks');
});
```

Use the actual Expo template for Babel, Metro, and Jest configuration. It keeps the native dependency versions aligned with its Expo SDK.
