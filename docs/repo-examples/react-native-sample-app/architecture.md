# Architecture — taskrabbit/ReactNativeSampleApp

Canonical URL: https://github.com/taskrabbit/ReactNativeSampleApp
Branch used for this doc: `master` (default branch).

This app is deliberately architecture-first: a Twitter/Tumblr-like social demo
(users, posts, follows) whose value is the **clean separation of concerns**, not the
features. Everything below maps to a real path verified in the repo.

## Repository layout (top level)

```
.babelrc  .buckconfig  .flowconfig  .gitignore  .travis.yml  .watchmanconfig
MIT-LICENSE  README.md  package.json
index.android.js        # Android entry point
index.ios.js            # iOS entry point
App/                    # all application source (see below)
android/                # Android native project
ios/                    # iOS native project
server/                 # small Node server serving in-memory data
tasks/                  # build / task automation
test/                   # tests (Appium integration per .travis.yml)
screenshots/
```

## The `App/` tree — concern by concern

```
App/
  Root.js               # top-level component: picks LoggedIn vs LoggedOut
  Dispatcher.js         # single shared Flux dispatcher singleton
  Locale.js             # i18n: Locale.key(namespace, defaults)
  jsVersion.js
  Actions/              # action creators (call services, then dispatch)
    AppActions.js  AuthActions.js  FollowActions.js  PostActions.js
  Api/                  # network / data layer
    HTTPClient.js       # the only place that knows superagent + headers + errors
    Network.js          # in-flight request tracking (started/completed)
    AuthService.js  FollowService.js  PostService.js
  Stores/               # Flux stores (EventEmitter, register with Dispatcher)
    CurrentUserStore.js  EnvironmentStore.js  DebugStore.js
    PostListStore.js  LocalKeyStore.js  ...
  Models/               # thin classes wrapping parsed JSON (e.g. CurrentUser)
  Navigation/           # URL-driven routing + nav chrome
    Router.js  Routes.js  Navigator.js
    NavigationBar.js  NavigationButton.js  NavigationHeader.js  NavigationTitle.js
  Screens/              # one component per screen
    LogIn.js  SignUp.js  PostList.js  FollowList.js  CreatePost.js
    Settings.js  Loading.js
  Mixins/               # cross-cutting behavior shared via React.createClass mixins
    DispatcherListener.js  ListHelper.js  (KeyboardListener, NavigationListener, ...)
  Components/           # shared widgets (SegmentedControl, SimpleList, Button, ...)
  Constants/
    AppConstants.js     # action-type string constants
  Extensions/  Lib/  Locales/  Platform/  Root/  Vendor/
    Platform/Keychain.js  # native-backed secure token storage
    Root/{Launch,Launcher,LoggedIn,LoggedOut,TestRunner}.js
```

(Directory listings for `App/`, `App/Api`, `App/Actions`, `App/Navigation`, and
`App/Screens` were read directly from the GitHub tree; individual `Components`,
`Models`, and some `Stores` filenames beyond those named above were not all
enumerated — see the "Unverified" note at the end.)

## How concerns are separated

| Layer | Responsibility | Real path(s) |
|---|---|---|
| Screens / Components | Render UI, subscribe to stores, call actions | `App/Screens/*`, `App/Components/*` |
| Mixins | Cross-cutting wiring (store listeners, lists, nav) | `App/Mixins/*` |
| Actions | Call a service, then `Dispatcher.dispatch({actionType, ...})` | `App/Actions/*` |
| Services (Api) | Own endpoints + parse server JSON into a stable shape | `App/Api/{Post,Auth,Follow}Service.js` |
| HTTPClient | Sole owner of superagent, headers, auth token, error normalization | `App/Api/HTTPClient.js` |
| Network | Tracks in-flight requests for global UI feedback | `App/Api/Network.js` |
| Dispatcher | One shared Flux dispatcher instance | `App/Dispatcher.js` |
| Stores | Hold state, register with the dispatcher, emit `'change'` | `App/Stores/*` |
| Models | Wrap parsed data in a thin class | `App/Models/*` |
| Navigation | URL parse/build + nav chrome | `App/Navigation/*` |
| Constants | Action-type strings shared by actions and stores | `App/Constants/AppConstants.js` |

## Data flow (unidirectional Flux)

The README states the loop; the source confirms it:

```
Component                Action                 Service / HTTPClient        Dispatcher        Store              Component
---------                ------                 --------------------        ----------        -----              ---------
PostActions.fetchList -> PostService.fetchList -> client.get(...)       -> (on success)   -> register() cb  -> emitChange()
(Screens/PostList.js)    (Actions/PostActions)   (Api/PostService.js,        Dispatcher        switch on         change listener
                                                  Api/HTTPClient.js)         .dispatch(...)    actionType        re-renders
```

Concretely:

1. `App/Screens/PostList.js` (via the `ListHelper` mixin) calls
   `PostActions.fetchList(username, cb)`.
2. `App/Actions/PostActions.js` calls `PostService.fetchList`, and on success does
   `Dispatcher.dispatch({ actionType: AppConstants.POST_LIST_UPDATED, listProps })`.
3. `App/Api/PostService.js` calls `client.get("api/posts/" + username, ...)` and
   reshapes the response with `parsePosts`/`parsePost` before handing it back.
4. `App/Api/HTTPClient.js` adds headers + auth token, runs the request through
   superagent, wraps `Network.started/completed`, parses JSON, and normalizes errors.
5. A store registered on the shared dispatcher (e.g. `App/Stores/PostListStore.js`)
   matches `action.actionType` in a `switch`, updates its in-memory data, and calls
   `emitChange()`.
6. Components added a change listener (commonly through the `DispatcherListener`
   mixin or `Store.addChangeListener`) and re-render. See `App/Root.js`, which wires
   `CurrentUserStore`/`EnvironmentStore`/`DebugStore` listeners in
   `componentDidMount` and dispatches `AppActions.appLaunched()`.

The store side of the loop is visible in `App/Stores/CurrentUserStore.js`: it is an
`EventEmitter` that calls `Dispatcher.register(fn)` and `switch`es on
`AppConstants.APP_LAUNCHED`, `LOGIN_USER`, `USER_UPDATED`, `LOGOUT_REQUESTED`.

## Navigation: URL is the single source of truth

The README: "the sole method of navigation ... is via urls." Implementation lives in
`App/Navigation/Router.js` (parse/build a path) and `App/Navigation/Routes.js`
(map path segments to screen components).

`Routes.parse(str, loggedIn, defaulted)` (in `App/Navigation/Routes.js`):

- Picks a `LoggedIn` vs `LoggedOut` route table based on auth state.
- Delegates to `Router.parse(...)`; falls back to `dashboard` (logged in) or
  `signup` (logged out) when nothing matches and a default is requested.
- Routes are **recursive**: `userRoute(username)` parses `posts` / `follows`, and a
  `follows` entry resolves each follow back into another `userRoute(follow)`. This is
  what lets a URL encode a whole stack like a friend's follower's posts.
- `listRoute` attaches sub-paths (`_post` -> `CreatePost`, `_settings` -> `Settings`)
  and nav-bar buttons, so the route definition also describes the nav chrome.

`App/Root.js` ties navigation to state: on user change it resets `navigationState`,
and on launch it can restore `debug.currentRoutePath` (simulator only) — i.e. the
app can boot directly into a saved deep route.

## Environment & storage

- `App/Stores/EnvironmentStore.js` exposes `getApiHost()`, which `HTTPClient.url()`
  uses to build every request URL — so the API host is environment-driven, not
  hardcoded in services.
- `App/Stores/LocalKeyStore.js` persists JSON locally; `CurrentUserStore` uses it
  plus `App/Platform/Keychain.js` to store the auth token securely.

## Unverified

- The full contents of `App/Components/`, `App/Models/`, `App/Extensions/`,
  `App/Lib/`, `App/Locales/`, and the complete `App/Stores/` file list were not all
  individually enumerated; names shown above are those confirmed in source or
  directory listings. The recursive git-tree API
  (`/git/trees/master?recursive=1`) returned HTTP 403, so the tree was reconstructed
  from per-directory GitHub listings and fetched files rather than one authoritative
  tree dump.
