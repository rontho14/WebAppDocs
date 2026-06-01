# React Native Sample App — Code Snippets (modern adaptation)

TaskRabbit's clean separation (isolated network layer, unidirectional flow,
URL-as-navigation), rewritten on our stack — **TypeScript + axios + React Query +
Redux Toolkit + React Navigation**. Each snippet cites the source pattern; the code
is the modern equivalent, not the ~2015 `flux`/`superagent`/`createClass` source.
Follow the links for the originals.

---

## 1. Single API client (was `App/Api/HTTPClient.js` over superagent)
*Pattern from `App/Api/HTTPClient.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Api/HTTPClient.js)*

```ts
// src/services/apiClient.ts — the one place that knows host, headers, auth
import axios from 'axios';
import { env } from '@/config/env';

export const apiClient = axios.create({ baseURL: env.apiUrl, timeout: 10_000 });

apiClient.interceptors.request.use((cfg) => {
  cfg.headers['X-Client-Version'] = env.appVersion;
  const token = getAuthToken();                       // from secure storage
  if (token) cfg.headers.Authorization = `Bearer ${token}`;
  return cfg;
});
```
Swap endpoints, add a device id, or change auth in exactly one file; UI untouched.

## 2. Error normalization (was the `wrapper` `{status, message, errors}` shaper)
*Pattern from `App/Api/HTTPClient.js` `wrapper` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Api/HTTPClient.js)*

```ts
// src/services/apiClient.ts
export type AppError = { status: number; message: string; errors: string[] };

apiClient.interceptors.response.use(
  (r) => r,
  (err): Promise<never> => {
    const e: AppError = {
      status: err.response?.status ?? 520,           // 520 = Unknown, same as the original
      message: err.response?.data?.error ?? `Server Error: ${err.response?.status ?? 520}`,
      errors: err.response?.data?.errors ?? [],
    };
    return Promise.reject(e);                          // every caller sees one shape
  },
);
```

## 3. In-flight tracking for a global indicator (was `Network.started/completed`)
*Pattern from `App/Api/Network.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Api/Network.js)*

```ts
// With React Query this is free — no manual counter needed:
// src/components/NetworkActivity.tsx
export function NetworkActivity() {
  const fetching = useIsFetching();                    // # of in-flight queries
  return fetching > 0 ? <ActivityIndicator /> : null;
}
```

## 4. Service owns endpoint + normalizes JSON (was `App/Api/PostService.js`)
*Pattern from `App/Api/PostService.js` (`parsePost`/`parsePosts`) — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Api/PostService.js)*

```ts
// src/services/postService.ts
export type Post = { id: string; content: string; username: string };
const toPost = (r: any): Post => ({ id: r.id, content: r.content, username: r.username });

export const postService = {
  list: (username: string) =>
    apiClient.get(`/posts/${username}`).then((r) => r.data.map(toPost)),
  create: (content: string) =>
    apiClient.post('/posts', { content }).then((r) => toPost(r.data)),
};
```

## 5. Unidirectional flow (was Action → Dispatcher → EventEmitter Store → Component)
*Pattern from `App/Actions/PostActions.js` + `App/Stores/*` — [actions](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Actions/PostActions.js) · [store](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Stores/CurrentUserStore.js)*

```ts
// src/features/posts/usePosts.ts — React Query is the modern unidirectional loop
export const usePosts = (username: string) =>
  useQuery({ queryKey: ['posts', username], queryFn: () => postService.list(username) });

export const useCreatePost = () => {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: postService.create,
    onSuccess: (post) => qc.invalidateQueries({ queryKey: ['posts', post.username] }),
  });
};
```
Component → mutation → service → cache invalidation → re-render: same loop the Flux
dispatcher/store did, with no hand-wired event bus.

## 6. Client/session state in a slice (was the `flux` `Dispatcher` + `CurrentUserStore`)
*Pattern from `App/Dispatcher.js` + `App/Stores/CurrentUserStore.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Stores/CurrentUserStore.js)*

```ts
// src/store/sessionSlice.ts — replaces the singleton EventEmitter store
const sessionSlice = createSlice({
  name: 'session',
  initialState: { user: null as User | null },
  reducers: {
    login: (s, a: PayloadAction<User>) => { s.user = a.payload; },
    logout: (s) => { s.user = null; },
  },
});
export const { login, logout } = sessionSlice.actions;
```

## 7. Screen: store-backed, no HTTP (was `App/Screens/PostList.js` + `ListHelper` mixin)
*Pattern from `App/Screens/PostList.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Screens/PostList.js)*

```tsx
// src/features/posts/PostListScreen.tsx — mixin behavior is now a hook
export function PostListScreen({ username }: { username: string }) {
  const { data: posts = [], isLoading, refetch } = usePosts(username);  // was ListHelper
  return (
    <FlatList
      data={posts}
      keyExtractor={(p) => p.id}
      refreshing={isLoading}
      onRefresh={refetch}
      renderItem={({ item }) => <Text>{item.content}</Text>}
    />
  );
}
```
No networking, no parsing — a dumb view over query state. The core TaskRabbit lesson.

## 8. URL is the single source of truth (was recursive `App/Navigation/Routes.js`)
*Pattern from `App/Navigation/Routes.js` / `Router.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Navigation/Routes.js)*

```ts
// React Navigation linking — declarative, nested routes encode the whole stack
const linking = {
  prefixes: ['kiosk://', 'https://app.example.com'],
  config: {
    screens: {
      Dashboard: 'dashboard',
      User: { path: 'user/:username', screens: { Posts: 'posts', Follows: 'follows' } },
    },
  },
};
// kiosk://user/john/follows boots directly there — deep-link / cold-restart into any screen
```

## 9. App root: bootstrap once, render from state (was `App/Root.js` store listeners)
*Pattern from `App/Root.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Root.js)*

```tsx
// src/App.tsx — subscribe via selector; swap tree on auth state
export default function App() {
  return (
    <ReduxProvider store={store}>
      <QueryClientProvider client={queryClient}>
        <NavigationContainer linking={linking}><RootSwitch /></NavigationContainer>
      </QueryClientProvider>
    </ReduxProvider>
  );
}
function RootSwitch() {
  const user = useAppSelector((s) => s.session.user);   // was CurrentUserStore listener
  return user ? <LoggedInNavigator /> : <LoggedOutNavigator />;
}
```

## 10. Environment-driven host + secure token (was `EnvironmentStore` + `Keychain`)
*Pattern from `App/Stores/EnvironmentStore.js` + `App/Platform/Keychain.js` — [source](https://github.com/taskrabbit/ReactNativeSampleApp/blob/master/App/Stores/EnvironmentStore.js)*

```ts
// src/config/env.ts — host from env, not hardcoded in services
import Config from 'react-native-config';
export const env = { apiUrl: Config.API_URL!, appVersion: Config.APP_VERSION ?? '1.0' };

// src/services/auth.ts — token in the Keychain, not plain storage
import * as Keychain from 'react-native-keychain';
export const saveToken = (t: string) => Keychain.setGenericPassword('auth', t);
export const getToken = async () =>
  (await Keychain.getGenericPassword()) ? (await Keychain.getGenericPassword() as any).password : null;
```

**Kept:** network layer behind one client, normalized errors, unidirectional flow,
URL-addressable screens, secure token. **Dropped:** `flux` dispatcher, superagent,
`React.createClass`, mixins, old `Navigator`.
