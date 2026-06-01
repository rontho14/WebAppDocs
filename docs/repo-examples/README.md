# Repo Examples — real open-source apps to learn from

Architecture and **verified** code snippets extracted from real open-source React
Native projects. Each subfolder documents one repo; code excerpts were fetched
from the live repositories with source paths + permalinks, and anything that
couldn't be verified is flagged in-doc.

> **For the AI agent:** open a repo's `architecture.md` to see how it's structured,
> `snippets.md` for copy-able verified code, and `README.md` for the stack +
> tablet/kiosk takeaways. These are **study references**, not our codebase —
> several are dated (see each doc's "dated-tech caveat"); adopt the *patterns*,
> not the obsolete APIs.

## Index
| Repo | Folder | Study it for |
|------|--------|--------------|
| **fbsamples/f8app** (F8 2017) | [`f8app/`](f8app/README.md) | Content-heavy multi-screen app; Redux data fan-out, reducer-boundary normalization, offline persistence, route-driven scenes |
| **RN UIExplorer / RNTester** | [`uiexplorer/`](uiexplorer/README.md) | Example-registry + list→detail pattern; self-contained feature modules; searchable/categorized component gallery |
| **taskrabbit/ReactNativeSampleApp** | [`react-native-sample-app/`](react-native-sample-app/README.md) | Clean separation of concerns; isolated network layer; unidirectional Flux; URL-as-navigation |

Each folder contains: `README.md` · `architecture.md` · `snippets.md`.

> ⚠️ These repos are dated (2015–2017). Before copying any code, read
> [`modern-equivalents.md`](modern-equivalents.md) — it maps each obsolete API
> (old `Navigator`, `ListView`, Flux, `React.createClass`) to our current stack
> (React Navigation, `FlatList`, Redux Toolkit, React Query, TypeScript).

## How these map to our project
- **F8** → how to structure a tablet/kiosk app that surfaces lots of content from a
  store and survives flaky networks (`f8app/`). Cross-ref: [`../01-architecture/state-management.md`](../01-architecture/state-management.md), [`../01-architecture/service-layer.md`](../01-architecture/service-layer.md).
- **UIExplorer/RNTester** → registry-driven menu/gallery of kiosk feature panels,
  with search and grouping (`uiexplorer/`). Cross-ref: [`../02-tablet-ux/navigation-menus.md`](../02-tablet-ux/navigation-menus.md).
- **TaskRabbit** → keeping the data/network layer behind one client and driving
  navigation from URLs for deep-link/restart (`react-native-sample-app/`).
  Cross-ref: [`../01-architecture/service-layer.md`](../01-architecture/service-layer.md).
