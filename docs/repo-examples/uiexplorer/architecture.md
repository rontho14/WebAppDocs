# UIExplorer / RNTester — Architecture

Two generations of the same idea: a **flat example registry** drives a
**list → detail** UI, and each example is a **self-describing module** rendered by
shared scaffolding. Paths below are real (verified against the source); see
`snippets.md` for the same patterns adapted to our current stack, each with a
source link.

---

## 1. Classic UIExplorer (`rnplay/uiexplorer`, branch `master`)

### Layout

The repo is mostly **flat** — every example is a single file at the repo root,
plus a handful of shared scaffolding files. Verified top-level contents:

```
ViewExample.js, TextExample.ios.js, ListViewExample.js, ScrollViewExample.js,
TouchableExample.js, ...                     # one module per component/API
ActionSheetIOSExample.js, AlertIOSExample.js, GeolocationExample.js, ...   # API examples
Navigator/                                   # nested example(s), e.g. NavigatorExample
UIExplorerList.js                            # the registry + list screen
createExamplePage.js                         # builds a detail page from a module
UIExplorerBlock.js                           # one titled/described example block
UIExplorerPage.js                            # page chrome
UIExplorerTitle.js
ExampleTypes.js                              # Flow types for example modules
index.ios.js                                 # AppRegistry root (UIExplorerApp)
package.json, .flowconfig
```

### The example-registry pattern

`UIExplorerList.js` declares the entire catalog as **two arrays of `require()`d
modules** — this *is* the registry:

```js
var COMPONENTS = [ require('./ViewExample'), require('./ListViewExample'), ... ];
var APIS       = [ require('./AlertIOSExample'), require('./GeolocationExample'), ... ];
```

(Note `require('./Navigator/NavigatorExample')` — registry entries can live in
subfolders; the array is the single source of truth, not the file layout.)

### How an example is declared

Each example module is a plain module exporting metadata + an `examples` array.
From `ViewExample.js`:

- `title` (e.g. `'<View>'`) and `description` (`'Basic building block of all UI.'`)
- `displayName` (used for snapshot test registration)
- `examples: [{ title, description?, render }]` — each `render` returns the demo JSX

### Discovery, navigation, search

All in `UIExplorerList.js`:

- **Discovery:** `COMPONENTS.concat(APIS)` is iterated everywhere a full catalog is
  needed (rendering rows, route lookup, snapshot registration).
- **List screen:** a `ListView` + `ListView.DataSource`
  (`ds.cloneWithRowsAndSections({components: COMPONENTS, apis: APIS})`). Section
  headers come from `_renderSectionHeader`; each row (`_renderRow`) shows
  `example.title` / `example.description`.
- **List → detail:** `_onPressRow` calls `makeRenderable(example)` then
  `this.props.navigator.push({ title, component })`.
- **Module → page:** `makeRenderable` returns `createExamplePage(null, example)`
  when the module has an `examples` array, else the module itself.
- **Search:** `_search(text)` builds a `RegExp` and filters both arrays by `title`,
  re-cloning the DataSource — a live filter over the registry.
- **Deep-link by title:** `render()` looks up `COMPONENTS.concat(APIS).find(c => c.title === route)` to jump straight to one example.

### Shared scaffolding (the generic renderer)

- `createExamplePage.js` — takes an example module, asserts it has `examples`, and
  maps each entry to a `UIExplorerBlock` inside a `UIExplorerPage`. The host never
  hard-codes any example's UI; it renders data.
- `UIExplorerBlock.js` — renders one example: a title, optional description, and
  `this.props.children` (the rendered demo). `propTypes` are just
  `{title, description}`.
- `index.ios.js` — `AppRegistry.registerComponent('UIExplorerApp', () => UIExplorerApp)`;
  `UIExplorerApp` delegates rendering to `UIExplorerList`.

### Data flow (classic)

```
index.ios.js (AppRegistry root)
  └─ UIExplorerApp → UIExplorerList
       ├─ registry: COMPONENTS[] + APIS[]   (arrays of required modules)
       ├─ ListView rows  ──tap──▶ navigator.push({title, component})
       └─ makeRenderable(module)
             └─ createExamplePage(module)
                   └─ map module.examples[] ▶ UIExplorerBlock(title, description){ render() }
```

---

## 2. Modern RNTester (`facebook/react-native`, branch `main`, `packages/rn-tester`)

### Layout (verified top level of `packages/rn-tester`)

```
js/            # all JS: examples, registry, types, navigation
android/app
RNTester/      # iOS app shell + RNTesterPods.xcodeproj/.xcworkspace
NativeComponentExample/, NativeModuleExample/, NativeCxxModuleExample/
IntegrationTests/, RNTester*Tests/, RCTTest/
.maestro/      # Maestro E2E flows
README.md, package.json, metro.config.js, Podfile, Gemfile, ...
```

Key `js/` subpaths used here:

```
js/utils/RNTesterList.ios.js      # the registry (also .android.js, .js.flow, FbInternal)
js/utils/RNTesterNavigationReducer.js
js/types/RNTesterTypes.js         # RNTesterModule / RNTesterModuleExample types
js/examples/View/ViewExample.js   # example module (and one folder per example)
```

### The evolved registry

`js/utils/RNTesterList.ios.js` keeps the same two-bucket idea but each entry is a
typed `RNTesterModuleInfo` `{ key, module, category? }`:

```js
{ key: 'ImageExample', module: require('../examples/Image/ImageExample'), category: 'Basic' }
```

Exported as `{ APIs, Components, Modules }`, where `Modules` is a key→module map
across all buckets (used for lookup/deep-linking). Examples now live in
**per-component folders** under `js/examples/`, not flat at the root.

### The typed example contract

`js/types/RNTesterTypes.js` formalizes what classic UIExplorer did informally:

- `RNTesterModule`: `{ title, description, examples: RNTesterModuleExample[], displayName?, documentationURL?, category?, showIndividualExamples? }`
- `RNTesterModuleExample`: `{ title, name?, description?, platform?, hidden?, scrollable?, render: component() }`

`ViewExample.js` exports `export default { ... } as RNTesterModule` with
`title: 'View'`, `category: 'Basic'`, `documentationURL`, and an `examples[]` of
`{title, name, render}` — each render carries `testID`s for E2E/Maestro tests.

### What changed vs. classic

| Concern | Classic UIExplorer | Modern RNTester |
|---|---|---|
| Registry entry | bare `require('./X')` | `{key, module, category}` (`RNTesterModuleInfo`) |
| Example layout | flat root `*Example.js` | `js/examples/<Name>/<Name>Example.js` |
| Typing | informal (`ExampleTypes.js`) | `RNTesterModule` / `RNTesterModuleExample` |
| Component style | `React.createClass`, `PropTypes` | function components, Flow `component()` |
| List UI | `ListView` + `DataSource` | modern list + `RNTesterNavigationReducer` |
| Per-example test hooks | none | `name`, `testID`, `.maestro/` flows |

The constant across both: **add a module to one registry array and the whole
gallery (list, detail, search, deep-link) picks it up** — the lesson to carry into
a tablet/kiosk component gallery.
