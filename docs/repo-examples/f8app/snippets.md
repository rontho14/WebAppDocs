# F8 App 2017 — Code Snippets

Real excerpts copied from files fetched from branch `main`. Each is trimmed to the
load-bearing lines; license headers removed for brevity. Permalinks point at the
`main` branch on GitHub.

## 1. One-line native entry point

**Source:** `index.ios.js` (identical `index.android.js`)
**URL:** https://github.com/fbsamples/f8app/blob/main/index.ios.js

```js
import { AppRegistry } from "react-native";
import setup from "./js/setup";

AppRegistry.registerComponent("F82017", setup);
```

The platform entry files do nothing but register `js/setup`. All real bootstrapping
is in JS, so a kiosk build keeps one code path across platforms.

## 2. Bootstrap: gate first paint on store rehydration

**Source:** `js/setup.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/setup.js

```js
componentDidMount() {
  configureStore(
    _ => this.setState({ storeRehydrated: true })
  ).then(
    store => this.setState({ store, storeCreated: true })
  );
}

render() {
  if (!this.state.storeCreated || !this.state.storeRehydrated) {
    return <LaunchScreen />;
  }
  return (
    <Provider store={this.state.store}>
      <F8App />
    </Provider>
  );
}
```

The app shows a `LaunchScreen` until the persisted store is rebuilt and rehydrated,
then mounts under react-redux `Provider`. A kiosk reboots straight into last-known
content instead of an empty UI.

## 3. Store config: middleware chain + persistence

**Source:** `js/store/configureStore.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/store/configureStore.js

```js
const createF8Store = applyMiddleware(thunk, promise, array, analytics, logger)(
  createStore
);

async function configureStore(onComplete) {
  const didReset = await ensureCompatibility();
  const store = autoRehydrate()(createF8Store)(reducers);
  persistStore(store, { storage: AsyncStorage }, _ => onComplete(didReset));
  return store;
}
```

Thunk plus three app-specific middlewares (`promise`, `array`, `analytics`), with
`redux-persist` writing through to `AsyncStorage`. The offline-first persistence
model a kiosk wants.

## 4. Root component fans out content loaders

**Source:** `js/F8App.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/F8App.js

```js
componentDidMount() {
  AppState.addEventListener("change", this.handleAppStateChange);
  this.props.dispatch(loadSessions());
  this.props.dispatch(loadConfig());
  this.props.dispatch(loadNotifications());
  this.props.dispatch(loadVideos());
  this.props.dispatch(loadMaps());
  this.props.dispatch(loadFAQs());
  this.props.dispatch(loadPages());
  this.props.dispatch(loadPolicies());
  // ...logged-in extras...
}

handleAppStateChange = appState => {
  if (appState === "active") {
    this.props.dispatch(loadSessions());
    this.props.dispatch(loadVideos());
    this.props.dispatch(loadNotifications());
  }
};
```

Pre-warm all content at boot, then refresh a subset whenever the app returns to the
foreground — exactly the loop a long-running kiosk screen wants.

## 5. Generic Parse loader thunk (data fetched off the animation frame)

**Source:** `js/actions/parse.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/actions/parse.js

```js
function loadParseQuery(type, query) {
  return dispatch => {
    return query.find({
      success: list => {
        InteractionManager.runAfterInteractions(() => {
          dispatch(({ type, list }: any));
        });
      },
      error: logError
    });
  };
}

function loadSessions() {
  return loadParseQuery(
    "LOADED_SESSIONS",
    new Parse.Query("Agenda").include("speakers").ascending("startTime")
  );
}
```

One reusable thunk for every read-only content type; dispatch is deferred behind
`InteractionManager` so loading never janks a transition.

## 6. Reducer factory normalizes backend objects

**Source:** `js/reducers/createParseReducer.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/reducers/createParseReducer.js

```js
function createParseReducer(type, convert) {
  return function(state, action) {
    if (action.type === type) {
      return action.list.map(convert);
    }
    return state || [];
  };
}
```

The single place Parse object shapes become plain UI state. Swap the data source and
only `convert` changes — components are untouched.

## 7. Root reducer: many small slices

**Source:** `js/reducers/index.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/reducers/index.js

```js
module.exports = combineReducers({
  config: require("./config"),
  sessions: require("./sessions"),
  maps: require("./maps"),
  faqs: require("./faqs"),
  pages: require("./pages"),
  videos: require("./videos"),
  navigation: require("./navigation"),
  // ...~20 slices total...
});
```

Each content area is an independent, individually testable slice — a clean template
for per-screen kiosk content.

## 8. Navigation kept in Redux

**Source:** `js/reducers/navigation.js` + `js/actions/navigation.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/reducers/navigation.js

```js
// actions/navigation.js
switchTab: (tab) => ({ type: "SWITCH_TAB", tab }),
switchDay: (day) => ({ type: "SWITCH_DAY", day }),

// reducers/navigation.js
const initialState = { tab: "schedule", day: 1 };
function navigation(state = initialState, action) {
  if (action.type === "SWITCH_TAB") return { ...state, tab: action.tab };
  if (action.type === "SWITCH_DAY") return { ...state, day: action.day };
  if (action.type === "LOGGED_OUT") return initialState;
  return state;
}
```

The selected tab/day is store state, so any component can read or change it — handy
for a kiosk that drives navigation from external events (idle reset, deep links).

## 9. Stack navigator: route-shape → scene

**Source:** `js/F8Navigator.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/F8Navigator.js

```js
renderScene: function(route, navigator) {
  if (route.allSessions) {
    return <SessionsCarousel {...route} navigator={navigator} />;
  } else if (route.session) {
    return <SessionsCarousel session={route.session} navigator={navigator} />;
  } else if (route.video) {
    return <F8VideoView video={route.video} navigator={navigator} />;
  } else if (route.maps) {
    return <F8MapView directions={false} navigator={navigator} />;
  } else {
    return <F8TabsView navigator={navigator} />;  // default: the tab shell
  }
}
```

A single declarative route → scene map; the route object's keys act as its type. A
multi-screen kiosk gets one place to define every detail/modal screen.

## 10. Tab bar driven by store state

**Source:** `js/tabs/F8TabsView.js`
**URL:** https://github.com/fbsamples/f8app/blob/main/js/tabs/F8TabsView.js

```js
<TabNavigator tabBarStyle={styles.tabBar}>
  <TabNavigator.Item
    title="Schedule"
    selected={this.props.tab === "schedule"}
    onPress={this.onTabSelect.bind(this, "schedule")}
    renderIcon={_ => this.renderTabIcon(scheduleIcon)}
  >
    <GeneralScheduleView now={this.state.now} navigator={this.props.navigator} />
  </TabNavigator.Item>
  {/* ...My F8, Demos, Videos, Info... */}
</TabNavigator>
```

The persistent tab shell: `selected` reads the Redux `tab`, `onPress` dispatches
`switchTab`. The home-level navigation of a content kiosk.
