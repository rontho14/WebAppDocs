# F8 App — Code Snippets (modern adaptation)

The F8 2017 patterns, rewritten on our current stack — **TypeScript + React Query +
Redux Toolkit + React Navigation + redux-persist v6**. Each snippet cites the
original source pattern it's adapted from; the code here is the modern equivalent
that does the same job (it intentionally does **not** match the 2017 source
line-for-line). For the original verbatim code, follow the source link.

---

## 1. Native entry point
*Pattern from `index.ios.js` / `js/setup.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/setup.js)*

```tsx
// index.js — one entry, all bootstrap in <App/>
import { AppRegistry } from 'react-native';
import App from './src/App';
import { name as appName } from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

## 2. Boot gate on rehydration (was the `LaunchScreen` until rehydrated check)
*Pattern from `js/setup.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/setup.js)*

```tsx
// src/App.tsx — PersistGate replaces the manual storeRehydrated flag
export default function App() {
  useEffect(() => { prefetchContent(queryClient); }, []);
  return (
    <ReduxProvider store={store}>
      <PersistGate loading={<LaunchScreen />} persistor={persistor}>
        <QueryClientProvider client={queryClient}>
          <NavigationContainer><RootNavigator /></NavigationContainer>
        </QueryClientProvider>
      </PersistGate>
    </ReduxProvider>
  );
}
```
Boots straight into last-known content, no empty UI — the F8 offline-launch behavior.

## 3. Store config + persistence (was `configureStore.js` + `autoRehydrate`)
*Pattern from `js/store/configureStore.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/store/configureStore.js)*

```ts
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { persistReducer, persistStore } from 'redux-persist';
import AsyncStorage from '@react-native-async-storage/async-storage';

const persisted = persistReducer({ key: 'ui', storage: AsyncStorage }, uiReducer);
export const store = configureStore({ reducer: { ui: persisted } }); // thunk + devtools built in
export const persistor = persistStore(store);
```

## 4. Offline-first content cache (was `redux-persist` over content slices)
*Pattern from `js/store/configureStore.js` persistence — [source](https://github.com/fbsamples/f8app/blob/main/js/store/configureStore.js)*

```ts
// src/app/queryClient.ts — persist the React Query cache instead of content in Redux
export const queryClient = new QueryClient({
  defaultOptions: { queries: { gcTime: 1000 * 60 * 60 * 24, retry: 2, staleTime: 60_000 } },
});
persistQueryClient({
  queryClient,
  persister: createAsyncStoragePersister({ storage: AsyncStorage }),
  maxAge: 1000 * 60 * 60 * 24, // yesterday's content survives a cold boot
});
```

## 5. Boot-time content fan-out (was `F8App.componentDidMount` dispatch batch)
*Pattern from `js/F8App.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/F8App.js)*

```ts
// src/app/prefetchContent.ts
export async function prefetchContent(qc: QueryClient) {
  await Promise.all([
    qc.prefetchQuery(sessionsQuery),
    qc.prefetchQuery(mapsQuery),
    qc.prefetchQuery(faqsQuery),
    qc.prefetchQuery(videosQuery),
  ]);
}
// refresh a subset when the kiosk returns to foreground (the old AppState handler)
useEffect(() => {
  const sub = AppState.addEventListener('change', (s) => {
    if (s === 'active') qc.invalidateQueries({ queryKey: ['sessions'] });
  });
  return () => sub.remove();
}, [qc]);
```

## 6. Generic content loader (was the `loadParseQuery` thunk)
*Pattern from `js/actions/parse.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/actions/parse.js)*

```ts
// src/services/sessionService.ts — async service replaces Parse.Query callbacks
export const sessionService = {
  list: () => apiClient.get('/sessions').then((r) => r.data.map(toSession)),
};
// src/features/schedule/useSessions.ts
export const sessionsQuery = { queryKey: ['sessions'] as const, queryFn: sessionService.list };
export const useSessions = () => useQuery(sessionsQuery);
```

## 7. Normalize remote data at the boundary (was `createParseReducer(convert)`)
*Pattern from `js/reducers/createParseReducer.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/reducers/createParseReducer.js)*

```ts
// src/services/sessionService.ts — the convert() job, now in the service
export type Session = { id: string; title: string; startTime: string; speakers: string[] };
const toSession = (raw: any): Session => ({
  id: raw.objectId ?? raw.id,
  title: raw.title,
  startTime: raw.startTime,
  speakers: (raw.speakers ?? []).map((s: any) => s.name),
});
```
Components see only `Session`; swap the backend and only `toSession` changes.

## 8. Nav/UI state in a slice (was the hand-written `navigation` reducer)
*Pattern from `js/reducers/navigation.js` + `js/actions/navigation.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/reducers/navigation.js)*

```ts
// src/store/uiSlice.ts
const uiSlice = createSlice({
  name: 'ui',
  initialState: { tab: 'schedule', day: 1 },
  reducers: {
    switchTab: (s, a: PayloadAction<string>) => { s.tab = a.payload; },
    switchDay: (s, a: PayloadAction<number>) => { s.day = a.payload; },
    resetOnLogout: () => ({ tab: 'schedule', day: 1 }),
  },
});
export const { switchTab, switchDay, resetOnLogout } = uiSlice.actions;
export default uiSlice.reducer;
```

## 9. Route-shape scenes → typed stack (was `F8Navigator.renderScene`)
*Pattern from `js/F8Navigator.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/F8Navigator.js)*

```tsx
export type RootStackParamList = {
  Tabs: undefined; Session: { sessionId: string }; Video: { videoId: string }; Map: undefined;
};
const Stack = createNativeStackNavigator<RootStackParamList>();
export const RootNavigator = () => (
  <Stack.Navigator>
    <Stack.Screen name="Tabs" component={TabsScreen} />
    <Stack.Screen name="Session" component={SessionScreen} />
    <Stack.Screen name="Video" component={VideoScreen} />
    <Stack.Screen name="Map" component={MapScreen} />
  </Stack.Navigator>
);
// navigation.navigate('Session', { sessionId })  — params are type-checked
```

## 10. Tab shell driven by state (was `F8TabsView` + `react-native-tab-navigator`)
*Pattern from `js/tabs/F8TabsView.js` — [source](https://github.com/fbsamples/f8app/blob/main/js/tabs/F8TabsView.js)*

```tsx
const Tab = createBottomTabNavigator();   // or a left nav rail on tablets
export const TabsScreen = () => (
  <Tab.Navigator>
    <Tab.Screen name="Schedule" component={ScheduleScreen} />
    <Tab.Screen name="MyF8" component={MyF8Screen} />
    <Tab.Screen name="Videos" component={VideosScreen} />
    <Tab.Screen name="Info" component={InfoScreen} />
  </Tab.Navigator>
);
```
On tablets prefer a persistent nav rail (see `../../02-tablet-ux/navigation-menus.md`).
