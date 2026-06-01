# React Native UIExplorer / RNTester

The official React Native example app that showcases every built-in component and
API in one browsable gallery. It is the canonical answer to "how does the React
Native team itself structure a component showcase?" — a flat **example registry**
feeding a **list → detail** navigation. Worth studying for any app that needs a
component gallery, a settings/admin screen, or a kiosk menu of self-contained
feature panels.

## Canonical repos + branches used for this doc

| Variant | Repo | Branch | What it is |
|---|---|---|---|
| Classic **UIExplorer** | `rnplay/uiexplorer` (mirror) | `master` | Early UIExplorer, flat `*Example.js` files at repo root. Verified via `raw.githubusercontent.com/rnplay/uiexplorer/master/<file>`. |
| Modern **RNTester** | `facebook/react-native` | `main` | The living successor at `packages/rn-tester`. Verified via `raw.githubusercontent.com/facebook/react-native/main/packages/rn-tester/<path>`. |

> The GitHub REST API (`api.github.com/.../contents` and `git/trees`) returned
> HTTP 403 (unauthenticated rate limit) during research, so directory listings
> were read from the GitHub web tree views and the original source was pulled from
> `raw.githubusercontent.com` to verify structure. `architecture.md` documents the
> real repos; `snippets.md` adapts their patterns to our current stack (with a
> source link per snippet).

## History: UIExplorer to RNTester

- **UIExplorer** lived inside `facebook/react-native` at `Examples/UIExplorer`
  (the `rnplay/uiexplorer` mirror used here snapshots that early code). Examples
  were plain modules exporting `title` / `description` / `examples[]`, collected
  in two arrays (`COMPONENTS`, `APIS`) inside `UIExplorerList.js`.
- It was **renamed to RNTester** and moved to `packages/rn-tester`. The core idea
  is unchanged, but the registry became a typed list of `{key, module, category}`
  entries (`RNTesterModuleInfo`) and example modules now conform to the
  `RNTesterModule` Flow type (`packages/rn-tester/js/types/RNTesterTypes.js`).
- The naming evolved too: `UIExplorerBlock`/`UIExplorerPage`/`UIExplorerList` →
  `RNTester*` equivalents; iOS-only `*IOSExample` modules were merged into
  cross-platform examples (e.g. `SwitchIOSExample` → `SwitchExample`).

## Why it's worth studying

- It is **the** reference implementation of a component gallery, maintained by the
  core team and kept current on `main`.
- Each example is a fully **self-contained module** (data + render), so the host
  app needs zero knowledge of any individual example — a clean plugin boundary.
- It demonstrates two generations of the same pattern, so you can see what the
  team kept (the registry + list/detail) and what they hardened (typing,
  categories, per-example `name`/`testID` for automated testing).

## Tablet / kiosk takeaways

1. **Example-registry pattern for a component gallery.** A single flat array of
   modules (`COMPONENTS` + `APIS`, modern: `Components` + `APIs`) is the entire
   "menu." To add a kiosk feature panel you add one registry entry; navigation,
   search, and the detail view all pick it up automatically. See `architecture.md`.
2. **Self-contained feature modules.** Each example exports `title`, `description`
   and an `examples[]` of `{title, render}`. This is an ideal shape for kiosk
   "cards" or admin panels: the screen is described as data, rendered generically
   by shared scaffolding (`UIExplorerBlock` / `createExamplePage`).
3. **Searchable/filterable index.** `UIExplorerList._search` filters the registry
   by a regex over `title`. On a large kiosk menu, the same approach gives a
   "jump to feature" search for free over the registry, no per-screen wiring.
4. **Categories for grouping.** The modern `RNTesterModuleInfo.category` field
   (`'UI'`, `'Basic'`, ...) groups entries into sections — directly reusable for
   grouping kiosk functions (e.g. "Check-in", "Catalog", "Help") in one index.
5. **Test hooks baked into examples.** Modern examples carry `name` and `testID`
   per render (e.g. `testID="view-test-border"`), so a gallery doubles as an
   automated UI-test surface — valuable for unattended kiosk regression checks.

## Dated-tech caveat

The **classic UIExplorer** code is old React Native (circa RN ~0.x): it uses
`React.createClass`, `React.PropTypes`, `ListView` + `ListView.DataSource`,
`NavigatorIOS`, and many iOS-only `*IOSExample` modules — all **deprecated or
removed** in current React Native. Treat its *patterns* as instructive, not its
APIs. For runnable, current code use **RNTester on `main`** (function components,
Flow `component()` types, `FlatList`/`SectionList`-era navigation). Pin to a
specific commit if you copy code, since `main` moves.
