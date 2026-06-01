# F8 App 2017 — Architecture

All paths below were verified against the live repo on branch `main`. The JS app
lives entirely under `js/`; native projects are in `android/` and `ios/`; the
backend (Parse Server + GraphQL) is in `server/`.

## `js/` folder structure (verified)

```
js/
  setup.js                 # app bootstrap: init Parse + FB SDK, build store, render <Root>
  F8App.js                 # connected root component; fires data-load thunks on mount
  F8Navigator.js           # legacy Navigator; maps route-shape -> scene
  F8Analytics.js
  FacebookSDK.js
  PushNotificationsController.js
  Playground.js
  relay-environment.js     # Relay 1.x Environment (network + RecordSource store)
  env.js                   # serverURL, parseAppID, graphqlURL, version
  flow-lib.js
  actions/                 # action creators (mostly redux-thunk async)
  reducers/                # one reducer per content slice + combineReducers
  store/                   # configureStore + custom middleware
  common/                  # shared UI (F8Colors, F8Fonts, LaunchScreen, F8WebView, ...)
  tabs/                    # top-level sections, each its own subfolder
    F8TabsView.js          # the tab bar (react-native-tab-navigator)
    MenuItem.js
    schedule/  videos/  maps/  info/  notifications/  demos/
  login/                   # LoginScreen, LoginModal
  filter/                  # FilterScreen
  rating/                  # RatingScreen (post-session surveys)
  video/
```

(Subfolders confirmed via the GitHub tree for `js/`, `js/tabs/`, `js/store/`,
`js/reducers/`, `js/actions/`.)

## Bootstrap sequence

`index.ios.js` / `index.android.js` are one line each — they register the component
returned by `js/setup.js`:

```js
import setup from "./js/setup";
AppRegistry.registerComponent("F82017", setup);
```

`js/setup.js` then:
1. Initializes Parse (`Parse.initialize`, `Parse.serverURL = ${serverURL}/parse`)
   and the Facebook SDK.
2. Calls `configureStore(...)` and, until the store is both **created** and
   **rehydrated**, renders `<LaunchScreen />` instead of the app.
3. Once ready, wraps `<F8App />` in react-redux `<Provider store={...}>`.

This gates first paint on store rehydration — the kiosk-relevant "show last-known
content immediately on launch" behavior.

## State management (Redux)

**Store config — `js/store/configureStore.js`.** Redux 3 `createStore` with
`applyMiddleware(thunk, promise, array, analytics, logger)`. The three custom
middlewares (`./promise`, `./array`, `./analytics`) live alongside it in `js/store/`.
Persistence uses `redux-persist`'s `autoRehydrate()` enhancer plus
`persistStore(store, { storage: AsyncStorage }, callback)`. A `compatibility` module
(`js/store/compatibility.js`, imported as `ensureCompatibility`) can reset persisted
state across versions.

**Root reducer — `js/reducers/index.js`.** `combineReducers` over ~20 single-concern
slices: `config`, `notifications`, `maps`, `sessions`, `user`, `schedule`,
`scheduleTopics`, `scheduleFilter`, `faqs`, `pages`, `navigation`,
`friendsSchedules`, `surveys`, `videos`, `videoTopics`, `videoFilter`, `policies`,
`testEventDates`.

**Reducer factory — `js/reducers/createParseReducer.js`.** Many content slices are
read-only lists from Parse, so they share one higher-order reducer:
`createParseReducer(type, convert)` returns a reducer that, on a matching action,
maps `action.list` through a per-type `convert` function. This is where backend
object shapes are normalized into plain UI state.

**Navigation as state — `js/reducers/navigation.js`.** The selected `tab` and `day`
live in Redux (`SWITCH_TAB`, `SWITCH_DAY`), not in a navigator. Components read them
via `connect`. `LOGGED_OUT` resets to the initial tab.

## Actions

`js/actions/index.js` is a barrel that spreads ~11 action modules
(`parse`, `navigation`, `login`, `schedule`, `filter`, `notifications`, `config`,
`surveys`, `test`, `installation`, `video`) into one export object.

- **Sync actions** are plain object creators, e.g. `switchTab`/`switchDay` in
  `js/actions/navigation.js`.
- **Async / data loaders** are thunks. `js/actions/parse.js` defines a generic
  `loadParseQuery(type, query)` thunk that runs a `Parse.Query`, then dispatches
  `{ type, list }` inside `InteractionManager.runAfterInteractions(...)` so loading
  never blocks animations. `loadSessions`, `loadMaps`, `loadNotifications`,
  `loadFAQs`, `loadPages`, `loadVideos`, `loadPolicies` are thin wrappers over it.

## Data layer

Two data sources coexist:

1. **Parse Server (primary, runtime).** Configured in `js/setup.js`; queried in
   `js/actions/parse.js` via classic `Parse.Object.extend(...)` + `Parse.Query`. The
   backend runs from `server/` via `docker-compose` (`npm run server`).
2. **Relay / GraphQL.** `js/relay-environment.js` builds a Relay 1.x `Environment`
   with a `Network.create(fetchQuery)` that POSTs `{ query, variables }` to
   `graphqlURL`, backed by a `RecordSource`/`Store`. Build steps exist in
   `package.json` (`npm run graphql`, `npm run relay` against
   `server/graphql/src/schema`). This was the in-progress migration target the
   makeitopen.com tutorials cover.

## Navigation

Two layers:

- **Stack — `js/F8Navigator.js`.** Uses the legacy `Navigator` from
  `react-native-deprecated-custom-components`. `renderScene(route, navigator)`
  branches on which key the route object carries (`route.session`, `route.video`,
  `route.maps`, `route.webview`, `route.filter`, `route.login`, `route.rate`, ...)
  to render the matching full-screen component, defaulting to `<F8TabsView />`.
  `configureScene` picks the transition the same way (push-from-right for detail,
  float-from-bottom for modals). It also wires Android hardware back via
  `BackAndroid` and the React legacy context API
  (`getChildContext` / `childContextTypes`).
- **Tabs — `js/tabs/F8TabsView.js`.** Uses `react-native-tab-navigator`’s
  `<TabNavigator>` / `<TabNavigator.Item>` for the persistent bottom tab bar
  (Schedule, My F8, Demos, Videos, Info). The active tab comes from Redux
  (`store.navigation.tab`) and selecting one dispatches `switchTab`.

## Notable patterns

- **Connected root that owns data loading.** `js/F8App.js` dispatches a batch of
  `load*` thunks in `componentDidMount` and re-runs a subset on `AppState` "active".
- **Reducer-boundary normalization** via `createParseReducer` keeps components
  decoupled from the backend SDK shape.
- **Route-as-data scene switching** in `F8Navigator` — the route object’s keys are
  effectively the route "type".
- **Navigation kept in Redux** (tab/day) rather than in the navigator, so any
  component can read or change it.
- **Flow types throughout** (`@flow`, `type Action`, `type ThunkAction`,
  `type Tab`/`Day`).

## Could not verify

- Exact file lists inside `js/store/`, `js/reducers/`, and `js/actions/` beyond the
  files explicitly fetched/imported above — the GitHub tree API was rate-limited, so
  these are inferred from `import`/`require` statements in fetched files (each
  imported path was confirmed to exist where probed, e.g. `createParseReducer.js`).
- `server/` internals were not fetched; described only from `package.json` scripts
  and `docker-compose.yml` references.
