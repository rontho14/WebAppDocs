# F8 App 2017 (fbsamples/f8app)

The official Facebook F8 2017 conference app. A real, shipped React Native app
released as open source for learning. Worth studying because it is a complete,
content-heavy, multi-screen event app with a clean Redux data flow — close in
spirit to a tablet/kiosk app that surfaces sessions, maps, FAQs, and video.

- **Canonical URL:** https://github.com/fbsamples/f8app
- **Branch used for this doc:** `main` (default branch; repo archived read-only Sep 2023)
- **Companion tutorials:** https://makeitopen.com

## Stack (confirmed from `package.json`)

| Concern | Library | Version (per `package.json`) |
|---|---|---|
| UI | `react-native` | 0.44.0 |
| UI runtime | `react` | 16.0.0-alpha.6 |
| State | `redux` | ^3.6.0 |
| React bindings | `react-redux` | ^4.4.6 |
| Async actions | `redux-thunk` | ^2.1.0 |
| Persistence | `redux-persist` | ^4.0.0-beta1 |
| Logging | `redux-logger` | ^2.7.4 |
| Selectors | `reselect` | ^2.5.4 |
| GraphQL client | `react-relay` / `relay-runtime` | ^1.4.0 |
| Backend SDK | `parse` (Parse Server) | ^1.11.0 |
| FB auth | `react-native-fbsdk` | 0.6.0 |
| Tab nav | `react-native-tab-navigator` | ^0.3.3 |
| Stack nav | `react-native-deprecated-custom-components` (`Navigator`) | ^0.1.1 |
| Types | `flow-bin` | 0.42 |

Two data layers coexist:
- **Parse Server** is the primary runtime data source. The app calls `Parse.Query`
  in `js/actions/parse.js`, and reducers convert Parse objects into plain state
  (`js/reducers/createParseReducer.js`). A local Parse Server runs via
  `docker-compose` (`server/`, `npm run server`).
- **Relay/GraphQL** is wired up (`js/relay-environment.js`, `relay-compiler` build
  step, `server/graphql/`) — it was the in-progress migration target the
  makeitopen.com tutorials walk through.

## Why it's worth studying

- A genuinely complete event app: schedule, my-schedule, maps, FAQ, info pages,
  videos, notifications, surveys — all driven from a single normalized store.
- Clean separation: thunk actions fetch, "create*Reducer" factories normalize,
  components stay dumb.
- It shows two generations of Facebook data tooling side by side (Parse → Relay).

## Tablet / kiosk takeaways

1. **Content fan-out from one bootstrap.** On mount the app dispatches one batch of
   loaders (`loadSessions`, `loadMaps`, `loadFAQs`, `loadPages`, `loadPolicies`,
   `loadVideos`, ...) in `js/F8App.js`. A kiosk can pre-warm all its content the
   same way at boot so screens render instantly from the store, with no spinners
   mid-interaction.
2. **Normalize remote data at the reducer boundary.** `createParseReducer` +
   per-type `fromParse*` converters mean components never touch the backend SDK
   shape. Swap Parse for any API and the UI is untouched — exactly what you want
   when a kiosk content source changes.
3. **Persist + rehydrate for offline-first.** `redux-persist` (`autoRehydrate`,
   `persistStore` over `AsyncStorage`) keeps the last loaded content on disk, so a
   kiosk with flaky connectivity still shows yesterday's data on launch.
4. **One stack navigator, scene config by route shape.** `F8Navigator` picks a
   transition based on which key the route carries (`session`, `video`, `webview`,
   ...). A multi-screen kiosk benefits from this central, declarative route → scene
   mapping rather than per-screen navigation glue.
5. **Defer data work behind interactions.** Parse results are dispatched inside
   `InteractionManager.runAfterInteractions(...)` so data loading never janks an
   animation. Critical for kiosk UIs that must feel instant under the finger.

## Dated-tech caveat

This is a **2017 codebase**. Several APIs are obsolete: React Native 0.44, the
legacy `Navigator` (already shipped here as `react-native-deprecated-custom-components`),
Redux 3 + `createStore`/`applyMiddleware`, `redux-persist` v4 beta with
`autoRehydrate`, Relay Classic-era 1.x, and `React.createClass`-style components.
**Study the patterns — data fan-out, reducer-boundary normalization, persistence,
route-driven scene config — not the verbatim APIs.** Modern equivalents are Redux
Toolkit, React Navigation, redux-persist v6, and hooks-based components.
