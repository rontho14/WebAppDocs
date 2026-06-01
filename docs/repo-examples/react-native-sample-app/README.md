# React Native Sample App (taskrabbit/ReactNativeSampleApp)

An early TaskRabbit-built React Native sample app modeled on a Twitter/Tumblr-like
social platform (users, posts, follows). It is deliberately **architecture-first**:
the README states the focus is "architectural concepts" over features or styling —
"a working app in which we implement new ideas or those that have worked for us so
far." Worth studying because it shows a clean, hand-rolled separation of concerns
(Flux store/dispatcher, an isolated network/data layer, URL-driven navigation) on a
codebase small enough to read end to end.

- **Canonical URL:** https://github.com/taskrabbit/ReactNativeSampleApp
- **Branch used for this doc:** `master` (default branch)

## Stack / patterns it demonstrates (confirmed from README + source)

| Concern | Approach in the repo | Where |
|---|---|---|
| State management | **Flux** — a single shared dispatcher, event-emitting stores | `App/Dispatcher.js`, `App/Stores/*` |
| Dispatcher | `flux` package's `Dispatcher`, one shared singleton instance | `App/Dispatcher.js` |
| Action creators | Plain objects that call a service then `Dispatcher.dispatch` | `App/Actions/*` |
| Network / data layer | Service modules over a single `HTTPClient` built on **superagent** | `App/Api/*` |
| Models | Thin model classes wrapping parsed JSON | `App/Models/*` |
| Navigation | **URL-based** routing; URLs encode the whole nav stack recursively | `App/Navigation/Router.js`, `App/Navigation/Routes.js` |
| Code sharing | **Mixins** (`React.createClass`) plus a higher-order-component alternative | `App/Mixins/*` |
| i18n | Component-level key definitions via `Locale.key(...)` | `App/Locale.js`, used per screen |
| Backend | A small Node server serving **in-memory** data | `server/*` |
| Tests | Appium integration testing | `test/`, `.travis.yml` |

### The stated data flow (from the README)

> - Components use Actions
> - Actions tend to use the API Services and dispatch an event
> - Stores are listening to the events
> - Components add and remove listeners to the Stores

This is classic unidirectional Flux: `Component -> Action -> Service (HTTP) -> Dispatcher -> Store -> Component`.

### Navigation philosophy (from the README)

> the sole method of navigation is via urls

URLs represent the entire navigation stack recursively, enabling deep links such as
`/dashboard/follows/john/follows/sarah/posts`. Each screen can therefore load
independently from basic data in the URL.

## Why it's worth studying

- The whole Flux loop — dispatcher, actions, stores, listeners — is visible in a
  handful of tiny files, with no framework hiding it.
- The network layer is completely isolated: one `HTTPClient` owns headers, auth
  token injection, JSON parsing, and error normalization; service modules own
  endpoints and shape-parsing. UI never sees raw responses.
- Recursive URL routing is an unusually clean idea for an app of this age and maps
  directly to deep-linking / kiosk-restart needs.

## Tablet / kiosk takeaways

1. **Keep the network layer behind a single client.** `HTTPClient` is the only place
   that knows about superagent, auth headers, the API host, and error shape
   (`App/Api/HTTPClient.js`). A kiosk that must swap endpoints, inject device
   identifiers, or change auth touches exactly one file — the UI is untouched.
2. **Parse remote data into a fixed shape at the service boundary.** Each service
   (`PostService.parsePost` / `parsePosts`) converts the server JSON into a small,
   stable object before it ever reaches a store or component. Backend changes stay
   contained, which matters when a kiosk's content source evolves independently.
3. **Drive navigation from URLs so any screen can be reached cold.** Because the URL
   encodes the whole stack, a kiosk can deep-link or restart directly into a deep
   screen (`App/Navigation/Router.js`, `Routes.js`) instead of replaying taps.
4. **Centralize cross-cutting UI behavior in mixins/HOCs.** Store-listener wiring,
   nav-bar state, keyboard handling, and a spinner-loader are factored into
   reusable units (`App/Mixins/*`), so screens stay focused on their own data —
   useful when many kiosk screens share the same loading/refresh chrome.
5. **Track in-flight requests in one place for global UI feedback.** `HTTPClient`
   calls `Network.started()` / `Network.completed()` around every request
   (`App/Api/Network.js`), a single hook a kiosk can use to drive a global activity
   indicator or offline banner.

## Dated-tech caveat

This is a **very early (~2015) React Native codebase**. The specific APIs are
obsolete: components use `React.createClass` with **mixins**, lists use the old
`ListView` (`App/Mixins/ListHelper.js`), navigation is the pre-React-Navigation
custom `Navigator`/`Router` (`App/Navigation/*`), and state is **pre-Redux Flux**
with a hand-shared `flux` `Dispatcher` and `EventEmitter` stores. **Study the
architectural separation — data/network isolated from UI, unidirectional flow,
URL-as-navigation — not the verbatim APIs.** Modern equivalents are hooks/function
components, `FlatList`, React Navigation, and Redux Toolkit / Zustand.
