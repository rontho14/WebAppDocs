# UIExplorer / RNTester — Code Snippets (modern adaptation)

The example-registry → list/detail pattern, rewritten on our stack — **TypeScript +
React Navigation + `SectionList` + hooks**. Each snippet cites the source pattern;
the code is the modern equivalent (not the verbatim 2016 `ListView`/`createClass`
source). Follow the source links for the originals.

Result: add **one typed entry** and the menu, search, grouping, and detail routing
all pick it up — the UIExplorer/RNTester win, type-safe.

---

## 1. Typed registry (was `var COMPONENTS = [require('./X'), ...]`)
*Pattern from `UIExplorerList.js` / `RNTesterList.ios.js` — [classic](https://github.com/rnplay/uiexplorer/blob/master/UIExplorerList.js) · [modern](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/utils/RNTesterList.ios.js)*

```ts
// src/features/registry.ts
import type { ComponentType } from 'react';
export type FeatureEntry = {
  key: string;                                   // stable id (RNTester's `key`)
  title: string;
  description?: string;
  category: 'Flow' | 'Browse' | 'Help' | 'Admin'; // RNTester's `category`
  screen: ComponentType;
  testID?: string;                               // gallery doubles as an E2E surface
};

import { CheckinScreen } from './checkin/CheckinScreen';
import { CatalogScreen } from './catalog/CatalogScreen';

export const FEATURES: FeatureEntry[] = [
  { key: 'checkin', title: 'Check in', category: 'Flow', screen: CheckinScreen, testID: 'feature-checkin' },
  { key: 'catalog', title: 'Catalog', category: 'Browse', screen: CatalogScreen, testID: 'feature-catalog' },
];
```

## 2. Key → entry map for lookup/deep-link (was concat + `.find(c => c.title === route)`)
*Pattern from `UIExplorerList.js` render lookup / RNTester `Modules` — [source](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/utils/RNTesterList.ios.js)*

```ts
export const FEATURES_BY_KEY: Record<string, FeatureEntry> =
  Object.fromEntries(FEATURES.map((f) => [f.key, f]));
```

## 3. Self-describing feature module (the `{title, description, examples[]}` contract)
*Pattern from `ViewExample.js` + `RNTesterTypes.js` — [classic](https://github.com/rnplay/uiexplorer/blob/master/ViewExample.js) · [types](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/types/RNTesterTypes.js)*

```ts
// src/features/types.ts — the informal classic contract, now a TS type
export type FeatureExample = { title: string; name?: string; render: () => React.ReactElement };
export type FeatureModule = {
  title: string;
  description?: string;
  category?: string;
  examples: FeatureExample[];
};
```

## 4. A module conforming to the contract (was `exports.title / exports.examples`)
*Pattern from `ViewExample.js` — [source](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/View/ViewExample.js)*

```tsx
// src/features/catalog/CatalogExamples.tsx
export const catalogModule: FeatureModule = {
  title: 'Catalog',
  description: 'Browse available items.',
  category: 'Browse',
  examples: [
    { title: 'Grid', name: 'catalog-grid', render: () => <CatalogGrid /> },
    { title: 'List', name: 'catalog-list', render: () => <CatalogList /> },
  ],
};
```

## 5. Generic detail renderer (was `createExamplePage` + `UIExplorerBlock`)
*Pattern from `createExamplePage.js` + `UIExplorerBlock.js` — [page](https://github.com/rnplay/uiexplorer/blob/master/createExamplePage.js) · [block](https://github.com/rnplay/uiexplorer/blob/master/UIExplorerBlock.js)*

```tsx
// src/features/ExamplePage.tsx — host renders from data, knows no specific screen
export function ExamplePage({ module }: { module: FeatureModule }) {
  return (
    <ScrollView>
      {module.examples.map((ex) => (
        <View key={ex.name ?? ex.title} testID={ex.name} style={styles.block}>
          <Text style={styles.blockTitle}>{ex.title}</Text>
          {ex.render()}
        </View>
      ))}
    </ScrollView>
  );
}
```

## 6. Grouped + searchable index (was `ListView` + `_search` regex)
*Pattern from `UIExplorerList.js` `_search` / `_renderRow` — [source](https://github.com/rnplay/uiexplorer/blob/master/UIExplorerList.js)*

```tsx
// src/features/FeatureListScreen.tsx
export function FeatureListScreen() {
  const nav = useNavigation<NativeStackNavigationProp<RootStackParamList>>();
  const [query, setQuery] = useState('');

  const sections = useMemo(() => {
    const re = new RegExp(query, 'i');                          // same regex-over-title idea
    const hits = FEATURES.filter((f) => re.test(f.title));
    const byCat = groupBy(hits, (f) => f.category);            // category sections
    return Object.entries(byCat).map(([title, data]) => ({ title, data }));
  }, [query]);

  return (
    <>
      <TextInput value={query} onChangeText={setQuery} placeholder="Search"
                 accessibilityLabel="Search features" style={styles.search} />
      <SectionList
        sections={sections}
        keyExtractor={(item) => item.key}
        renderSectionHeader={({ section }) => <Text style={styles.header}>{section.title}</Text>}
        renderItem={({ item }) => (
          <Pressable testID={item.testID} onPress={() => nav.navigate('Feature', { key: item.key })}
                     style={{ minHeight: 56, justifyContent: 'center' }}>
            <Text>{item.title}</Text>
            {item.description ? <Text style={styles.sub}>{item.description}</Text> : null}
          </Pressable>
        )}
      />
    </>
  );
}
```

## 7. List → detail routing (was `navigator.push({ component })`)
*Pattern from `UIExplorerList._onPressRow` + `index.ios.js` — [source](https://github.com/rnplay/uiexplorer/blob/master/UIExplorerList.js)*

```tsx
export type RootStackParamList = { FeatureList: undefined; Feature: { key: string } };
const Stack = createNativeStackNavigator<RootStackParamList>();

function FeatureScreen({ route }: NativeStackScreenProps<RootStackParamList, 'Feature'>) {
  const entry = FEATURES_BY_KEY[route.params.key];
  if (!entry) return <Text>Unknown feature</Text>;
  const Screen = entry.screen;                  // generic render
  return <Screen />;
}

export const GalleryNavigator = () => (
  <Stack.Navigator>
    <Stack.Screen name="FeatureList" component={FeatureListScreen} options={{ title: 'Menu' }} />
    <Stack.Screen name="Feature" component={FeatureScreen}
      options={({ route }) => ({ title: FEATURES_BY_KEY[route.params.key]?.title })} />
  </Stack.Navigator>
);
```

## 8. Deep-link straight to a feature (was title lookup in `render()`)
*Pattern from `UIExplorerList.render` deep-link-by-title — [source](https://github.com/rnplay/uiexplorer/blob/master/UIExplorerList.js)*

```ts
// React Navigation linking config
const linking = {
  prefixes: ['kiosk://'],
  config: { screens: { FeatureList: 'menu', Feature: 'feature/:key' } },
};
// kiosk://feature/catalog opens that feature directly — handy for idle-reset/restart
```

**Kept:** one registry drives list + search + grouping + detail; self-describing
modules; per-entry test IDs. **Dropped:** `ListView`, `React.createClass`,
`require()` arrays, `NavigatorIOS`.
