# Repo Examples — real open-source apps to learn from

Architecture and code from real open-source React Native projects. Each subfolder
documents one repo: `architecture.md` describes the **real** repo (structure +
source paths/permalinks, with anything unverified flagged); `snippets.md` gives
the same patterns **rewritten on our current stack** (TypeScript + React
Navigation + Redux Toolkit + React Query) so the code is copy-ready, not dated.

> **For the AI agent:** open a repo's `architecture.md` to understand how it's
> structured, `snippets.md` for modern, copy-able code adapting its patterns, and
> `README.md` for the stack + tablet/kiosk takeaways. These are **study
> references**, not our codebase.

## Index
| Repo | Folder | Study it for |
|------|--------|--------------|
| **fbsamples/f8app** (F8 2017) | [`f8app/`](f8app/README.md) | Content-heavy multi-screen app; Redux data fan-out, reducer-boundary normalization, offline persistence, route-driven scenes |
| **RN UIExplorer / RNTester** | [`uiexplorer/`](uiexplorer/README.md) | Example-registry + list→detail pattern; self-contained feature modules; searchable/categorized component gallery |
| **taskrabbit/ReactNativeSampleApp** | [`react-native-sample-app/`](react-native-sample-app/README.md) | Clean separation of concerns; isolated network layer; unidirectional Flux; URL-as-navigation |

Each folder contains: `README.md` · `architecture.md` · `snippets.md`.

> ⚠️ The original repos are dated (2015–2017). `architecture.md` documents them
> as-is (old `Navigator`, `ListView`, Flux, `React.createClass`); the code in
> `snippets.md` is already modernized to our stack, so copy from there.

## How these map to our project
- **F8** → how to structure a tablet/kiosk app that surfaces lots of content from a
  store and survives flaky networks (`f8app/`). Cross-ref: [`../01-architecture/state-management.md`](../01-architecture/state-management.md), [`../01-architecture/service-layer.md`](../01-architecture/service-layer.md).
- **UIExplorer/RNTester** → registry-driven menu/gallery of kiosk feature panels,
  with search and grouping (`uiexplorer/`). Cross-ref: [`../02-tablet-ux/navigation-menus.md`](../02-tablet-ux/navigation-menus.md).
- **TaskRabbit** → keeping the data/network layer behind one client and driving
  navigation from URLs for deep-link/restart (`react-native-sample-app/`).
  Cross-ref: [`../01-architecture/service-layer.md`](../01-architecture/service-layer.md).
