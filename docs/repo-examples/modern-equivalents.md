# Modern Equivalents — dated patterns → current stack

The reference apps in this folder are from **2015–2017**. Their *architecture* is
worth copying; their *APIs* are not. This table maps what you'll see in
`f8app/`, `uiexplorer/`, and `react-native-sample-app/` to the stack we
standardize on. Keep the pattern, swap the API.

## Quick map
| Concern | Dated API (in the examples) | Use today |
|--------|------------------------------|-----------|
| Components | `React.createClass`, mixins | Function components + hooks |
| Cross-cutting reuse | Mixins (`DispatcherListener`, `ListHelper`) | Custom hooks / HOCs |
| Types | `React.PropTypes`, Flow | **TypeScript strict** ([`../05-quality/typescript-strict.md`](../05-quality/typescript-strict.md)) |
| Lists | `ListView` + `ListView.DataSource` | `FlatList` / `SectionList` (or FlashList) |
| Stack/scene nav | `Navigator`, `react-native-deprecated-custom-components`, custom `Router` | **React Navigation** (native stack) ([`../02-tablet-ux/navigation-menus.md`](../02-tablet-ux/navigation-menus.md)) |
| Tabs | `react-native-tab-navigator`, `NavigatorIOS` | React Navigation tab/drawer, or a tablet nav rail |
| Client state | Hand-rolled **Flux** (dispatcher + `EventEmitter` stores) | **Redux Toolkit** ([`../01-architecture/state-management.md`](../01-architecture/state-management.md)) |
| Server state / caching | Thunks writing into Redux; manual refetch | **React Query** ([`../01-architecture/service-layer.md`](../01-architecture/service-layer.md)) |
| Persistence | `redux-persist` v4 `autoRehydrate()` | `redux-persist` v6 (`persistReducer` + `PersistGate`) |
| GraphQL | Relay Classic 1.x (`Network.create`, `RecordSource`) | Modern Relay (hooks) or Apollo Client |
| Backend SDK calls | `Parse.Query(...).find({success, error})` callbacks | `async/await` services behind one API client |
| HTTP | `superagent` | `axios` (or `fetch`) in a single `apiClient` |
| URL routing | Custom recursive `Routes.parse` | React Navigation **linking** config (deep links) |
| Example registry | `require()`d module arrays | Typed registry (`import` + a `RegistryEntry` type) |
| Still valid | `InteractionManager`, `AppState`, `combineReducers` idea | Keep — these survived |

## Before → after (the high-value ones)

### Component + lifecycle
```tsx
// THEN (taskrabbit, f8app)
var PostList = React.createClass({
  mixins: [DispatcherListener],
  componentDidMount() { PostActions.fetchList(this.props.user); },
  render() { /* ListView */ },
});

// NOW
function PostList({ user }: { user: string }) {
  const { data } = usePosts(user);            // React Query hook
  return <FlatList data={data} renderItem={renderRow} />;
}
```

### State: Flux store → Redux Toolkit slice
```ts
// THEN: EventEmitter store + Dispatcher.register + switch(actionType)
// NOW:
const postsSlice = createSlice({
  name: 'posts',
  initialState: { byUser: {} as Record<string, Post[]> },
  reducers: { setPosts: (s, a) => { s.byUser[a.payload.user] = a.payload.list; } },
});
```
For server data prefer React Query over putting it in Redux at all.

### Server fetch: callbacks → async services + React Query
```ts
// THEN: new Parse.Query('Agenda').find({ success, error })
// NOW:
export const sessionService = {
  list: () => apiClient.get<Session[]>('/sessions').then((r) => r.data),
};
export const useSessions = () =>
  useQuery({ queryKey: ['sessions'], queryFn: sessionService.list, staleTime: 60_000 });
```
Keep F8's *pattern* — fan out loaders at boot, normalize at the boundary,
persist for offline — just with these APIs.

### Navigation: custom Navigator/route-shape → React Navigation
```tsx
// THEN: renderScene(route) { if (route.session) return <SessionsCarousel/> ... }
// NOW:
const Stack = createNativeStackNavigator<RootStackParamList>();
<Stack.Navigator>
  <Stack.Screen name="Session" component={SessionScreen} />
  <Stack.Screen name="Video" component={VideoScreen} />
</Stack.Navigator>
```

### URL-as-navigation → React Navigation linking
```ts
// THEN: Routes.parse('/dashboard/follows/john/posts') recursively
// NOW: declarative linking config = deep links + restore for free
const linking = {
  prefixes: ['kiosk://'],
  config: { screens: { Dashboard: 'dashboard', Profile: 'user/:id', Posts: 'user/:id/posts' } },
};
```
This preserves the examples' best idea (a route string can boot you into any
screen — ideal for kiosk restart/deep-link) on a supported API.

### Example registry: `require()` array → typed registry
```ts
// THEN: var COMPONENTS = [ require('./ViewExample'), require('./ListViewExample') ];
// NOW:
type RegistryEntry = { key: string; title: string; category: string; screen: React.ComponentType };
export const FEATURES: RegistryEntry[] = [
  { key: 'checkin', title: 'Check in', category: 'Flow', screen: CheckinScreen },
  { key: 'catalog', title: 'Catalog', category: 'Browse', screen: CatalogScreen },
];
```
Same win as UIExplorer/RNTester: add one entry and the menu, search, and
detail view pick it up — now type-safe.

## Rule of thumb
When copying from a reference app: **lift the boundary, not the byte.** Take the
separation (UI ↔ actions ↔ services ↔ client/store, registry-driven menus,
URL-addressable screens) and re-express it with TypeScript + React Navigation +
Redux Toolkit + React Query.
