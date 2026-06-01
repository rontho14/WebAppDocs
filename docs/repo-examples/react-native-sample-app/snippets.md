# Snippets — taskrabbit/ReactNativeSampleApp

Real, verbatim excerpts from files fetched from branch `master`. Each excerpt notes
its exact source path, a raw/permalink URL, and why it matters for a tablet/kiosk
app. Some excerpts are trimmed (comments removed / `...`) where noted; nothing is
invented.

> Dated-tech reminder: this is a ~2015 codebase (`React.createClass`, mixins, old
> `Navigator`, pre-Redux `flux`). Study the *separation*, not the verbatim APIs.

---

## 1. The single shared dispatcher

Source: `App/Dispatcher.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Dispatcher.js

```js
import flux from 'flux';
const {Dispatcher} = flux;

export default new Dispatcher();
```

One module exports one dispatcher instance the whole app imports. Demonstrates a
single, explicit event bus — every action and every store meet here. For a kiosk,
having one obvious choke point for all state changes makes the data flow auditable.

---

## 2. Action creator: call a service, then dispatch

Source: `App/Actions/PostActions.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Actions/PostActions.js

```js
import Dispatcher   from '../Dispatcher';
import AppConstants from '../Constants/AppConstants';
import PostService  from '../Api/PostService';

var PostActions = {
  fetchList: function(username, callback) {
    PostService.fetchList(username, function(error, listProps) {
      if(callback) callback(error);
      if (!error) {
        Dispatcher.dispatch({
          actionType: AppConstants.POST_LIST_UPDATED,
          listProps: listProps
        });
      }
    });
  },

  createPost: function(content, callback) {
    PostService.createPost(content, function(error, postProps) {
      if(callback) callback(error);
      if (!error) {
        Dispatcher.dispatch({
          actionType: AppConstants.POST_ADDED,
          postProps: postProps
        });
      }
    });
  }
};

export default PostActions;
```

Actions never touch HTTP directly — they call a service, and on success dispatch a
typed action. The UI calls `PostActions.fetchList(...)` and stays ignorant of the
network. A kiosk can reroute, retry, or mock the service without changing any screen.

---

## 3. Service: own the endpoint and parse into a stable shape

Source: `App/Api/PostService.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Api/PostService.js

```js
import client from '../Api/HTTPClient';

var PostService = {
  parsePost: function(response) {
    if (!response) return null;
    return {
      id: response.id,
      content: response.content,
      username: response.username
    };
  },

  fetchList: function(username, callback) {
    client.get("api/posts/" + username, {}, function(error, response) {
      var listProps = PostService.parsePosts(response);
      callback(error, listProps);
    });
  },

  createPost: function(content, callback) {
    client.post("api/posts", {content: content}, function(error, response) {
      var postProps = PostService.parsePost(response);
      callback(error, postProps);
    });
  },
};

export default PostService;
```

(Trimmed: `parsePosts` loop omitted.) The service is the *only* place that knows the
endpoint string and the server JSON shape. `parsePost` collapses the raw response
into a tiny fixed object before it reaches a store or component — so a backend change
is contained here, exactly what you want when a kiosk's content source evolves.

---

## 4. HTTPClient: the only place that knows superagent, headers, and errors

Source: `App/Api/HTTPClient.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Api/HTTPClient.js

```js
import superagent from 'superagent';
import Network from '../Api/Network';
import CurrentUserStore from '../Stores/CurrentUserStore';
import EnvironmentStore from '../Stores/EnvironmentStore';

var HTTPClient = {
  addHeaders: function(req) {
    var appVersion = "1.0";
    var userAgent = "Sample iPhone v" + appVersion;
    req = req.accept('application/json').type('application/json');
    req = req.set('User-Agent', userAgent);
    req = req.set('X-CLIENT-VERSION', appVersion);
    req = req.set('X-LOCALE', 'en-US');

    var currentUser = CurrentUserStore.get();
    if (currentUser && currentUser.getToken()) {
      req = req.set('Authorization', 'Bearer ' + currentUser.getToken());
    }
    return req;
  },

  fetch: function(req, callback) {
    req = this.addHeaders(req);
    Network.started();
    req.end(this.wrapper(callback));
  },

  url: function(path) {
    var host = EnvironmentStore.get().getApiHost();
    return host + "/" + path;
  },

  get: function(path, params, callback) {
    var req = superagent.get(this.url(path));
    if (params) { req = req.query(params); }
    this.fetch(req, callback);
  }
  // post / put similar
};

export default HTTPClient;
```

(Trimmed: `wrapper` error-normalizer, `post`, `put`, and some headers omitted.) Auth
token injection, the API host (from `EnvironmentStore.getApiHost()`), standard
headers, and in-flight tracking all live in one file. A kiosk that must swap
endpoints, add a device id, or change auth edits exactly one place; the UI never
changes.

---

## 5. Error normalization in one wrapper

Source: `App/Api/HTTPClient.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Api/HTTPClient.js

```js
wrapper: function(inner) {
  return function(error, response) {
    Network.completed();
    if(!inner) return;

    var parsed = null;
    if(response && response.text && response.text.length > 0) {
      try { parsed = JSON.parse(response.text); }
      catch (e) { parsed = null; }
    }

    var errorObj = null, valueObj = null;
    if (error) {
      errorObj = {};
      errorObj.status = error.status ? error.status : 520; // 520 = Unknown
      errorObj.errors = [];
      if (parsed && parsed.error) { errorObj.message = parsed.error; }
      if (!errorObj.message) { errorObj.message = 'Server Error: ' + errorObj.status; }
    }
    else { valueObj = parsed; }
    inner(errorObj, valueObj);
  };
},
```

(Trimmed: logging lines removed.) Every response passes through one normalizer that
guarantees the same `{status, message, errors}` error shape and always fires
`Network.completed()`. Callers handle one predictable error contract — a kiosk can
map that single shape to a global retry/offline banner.

---

## 6. Store: EventEmitter that registers with the dispatcher

Source: `App/Stores/CurrentUserStore.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Stores/CurrentUserStore.js

```js
import {EventEmitter} from 'events';
import assign   from 'object-assign';
import Dispatcher   from '../Dispatcher';
import AppConstants from '../Constants/AppConstants';

var CHANGE_EVENT = 'change';
var _singleton = null; // null so we know to initialize on app launch

var SingletonStore = assign({}, EventEmitter.prototype, {
  get: function() { return _singleton; },
  emitChange: function() { this.emit(CHANGE_EVENT); },
  addChangeListener: function(cb) { this.on(CHANGE_EVENT, cb); },
  removeChangeListener: function(cb) { this.removeListener(CHANGE_EVENT, cb); }
});

Dispatcher.register(function(action) {
  switch(action.actionType) {
    case AppConstants.LOGIN_USER:
    case AppConstants.USER_UPDATED:
      setUserProps(action.userProps, action.token);
      SingletonStore.emitChange();
      saveSingleton();
      break;
    case AppConstants.LOGOUT_REQUESTED:
      clearData();
      SingletonStore.emitChange();
      break;
    default: // no op
  }
});

export default SingletonStore;
```

(Trimmed: keychain/local-store helpers and `APP_LAUNCHED` branch omitted.) The store
holds state, registers a callback on the shared dispatcher, `switch`es on action
type, and emits `'change'`. This is the receiving half of the unidirectional loop —
the pattern Redux later formalized. Useful to see the mechanism with nothing hidden.

---

## 7. Mixin: subscribe a component to dispatched actions

Source: `App/Mixins/DispatcherListener.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Mixins/DispatcherListener.js

```js
import Dispatcher from '../Dispatcher';

var DispatcherListener = {
  unregisterDispatcher: function() {
    if(this.dispatchToken) {
      Dispatcher.unregister(this.dispatchToken);
      this.dispatchToken = null;
    }
  },
  dispatchedActionCallback: function(action) {
    if (this.isMounted()) {
      if (action.targetPath) {
        if (this.props.currentRoute && this.props.currentRoute.routePath === action.targetPath) {
          this.dispatchAction(action);
        }
      } else {
        this.dispatchAction(action);
      }
    }
  },
  componentDidMount: function() {
    this.unregisterDispatcher();
    this.dispatchToken = Dispatcher.register(this.dispatchedActionCallback);
  },
  componentWillUnmount: function() {
    this.unregisterDispatcher();
  },
};

export default DispatcherListener;
```

Cross-cutting subscription/teardown lives once in a mixin and registers/unregisters
on mount/unmount, with route-target filtering built in. The modern equivalent is a
hook, but the lesson holds: factor shared lifecycle wiring out of screens so each
kiosk screen only declares its own data needs.

---

## 8. Screen: declarative, store-backed, no HTTP

Source: `App/Screens/PostList.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Screens/PostList.js

```js
import React from 'react';
import ListHelper    from '../Mixins/ListHelper';
import PostListStore from '../Stores/PostListStore';
import PostActions   from '../Actions/PostActions';

var PostList = React.createClass({
  mixins: [ListHelper],

  getListItems: function() {
    return PostListStore.get(this.getUsername());
  },
  getItemProps: function(post) {
    return { key: post.data.id, title: post.data.content };
  },
  reloadList: function() {
    PostActions.fetchList(this.getUsername(), function(error) {
      if (error) { console.log(error.message); }
    });
  }
});

export default PostList;
```

(Trimmed: `getDefaultProps` segmented-control config omitted.) The screen reads from
`PostListStore`, asks `PostActions` to reload, and maps items to props — it has no
networking, no parsing, no navigation logic. This data/UI cleanliness is the core
takeaway for kiosk screens: dumb views over a store, fed by actions/services.

---

## 9. URL-driven, recursive routing

Source: `App/Navigation/Routes.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Navigation/Routes.js

```js
var userRoute = function(username) {
  var route = {};
  route._notAddressable = true;
  route._routerAppend = 'posts';

  route.parse = function(path) {
    switch(path) {
      case 'posts':
        return listRoute(Routes.PostList(username), function(postId) {
          return null;
        });
      case 'follows':
        return listRoute(Routes.FollowList(username), function(follow) {
          return userRoute(follow); // a follow resolves into another user route
        });
      default:
        return null;
    };
  };
  return route;
};

export default {
  parse: function(str, loggedIn, defaulted) {
    var parent = loggedIn ? LoggedIn : LoggedOut;
    var found = Router.parse(str, parent, defaulted);
    if (!found && defaulted) {
      found = loggedIn ? this.parse('dashboard', true, false)
                       : this.parse('signup', false, false);
    }
    return found;
  }
};
```

(Trimmed: `Routes` factory map and `listRoute` body omitted.) The URL is the single
source of truth and routes are recursive — a `follows` segment resolves each follow
back into a full `userRoute`, so one URL encodes an entire stack. For a kiosk this
means deep-linking and cold restart into any screen without replaying taps.

---

## 10. Root: tie store listeners to lifecycle, kick off on launch

Source: `App/Root.js`
Raw: https://raw.githubusercontent.com/taskrabbit/ReactNativeSampleApp/master/App/Root.js

```js
componentDidMount: function() {
  CurrentUserStore.addChangeListener(this.onUserChange);
  EnvironmentStore.addChangeListener(this.onEnvChange);
  DebugStore.addChangeListener(this.onDebugChange);
  AppActions.appLaunched();
},

componentWillUnmount: function() {
  DebugStore.removeChangeListener(this.onDebugChange);
  EnvironmentStore.removeChangeListener(this.onEnvChange);
  CurrentUserStore.removeChangeListener(this.onUserChange);
},

renderContent: function() {
  // ... pick navigationState from saved route / default
  if (this.state.user.isLoggedIn()) {
    return (<LoggedIn ref="current" navigationState={navigationState} />);
  } else {
    return (<LoggedOut ref="current" navigationState={navigationState} />);
  }
}
```

(Trimmed: `getInitialState`, route-restore logic, and the `test`-env branch omitted.)
The root component subscribes to the user/environment/debug stores, fires
`AppActions.appLaunched()` once to bootstrap, and swaps the whole tree between
logged-in/out based on store state. A kiosk's single bootstrap-then-render entry
point can follow the same shape: load config/state once, then render from stores.
