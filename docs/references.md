# References

Primary sources behind this knowledge base. Citations in the docs point here.

## Android (official)
- **Adaptive / Large-Screen App Quality guidelines** — the Tier 1/2/3 checklist,
  config & continuity, multi-window, input, and the test-device matrix.
  `developer.android.com/docs/quality-guidelines/.../large-screen-app-quality`
- **Large screens guide** — window size classes (Compact/Medium/Expanded) and
  canonical layouts (List-Detail, Feed, Supporting Pane).
  `developer.android.com/guide/topics/large-screens`
- **Window size classes** — full width breakpoints (Compact <600, Medium 600–840,
  Expanded 840–1200, Large 1200–1600, Extra-large ≥1600) + height classes
  (<480 / 480–900 / ≥900). `developer.android.com/develop/ui/compose/layouts/adaptive/window-size-classes`
- **Canonical layouts** — list-detail behavior per width; feed adaptive columns;
  supporting-pane proportions (70/30 expanded, 50/50 medium).
  `developer.android.com/develop/ui/compose/layouts/adaptive/canonical-layouts`
- **Android Design** — adaptive layouts, navigation rail vs bottom nav, and the
  three quality pillars: accessibility, technical quality, security & privacy.
  `developer.android.com/design`
- **"Helping Users Discover Quality Apps on Large Screens"** (Android Developers
  Blog, 2022) — Play Store ranks/features apps by large-screen quality
  (orientation + keyboard support), device-specific ratings/reviews, and
  low-quality alerts. `android-developers.googleblog.com/2022/03/...`

## UX research
- **NN/g — "Touch Targets on Touchscreens"** — ~1 cm physical target for adult
  fingertips. `nngroup.com/articles/touch-target-size/`
- **NN/g — "Tablet Usability: Findings from User Research"** & the *Tablet
  Website and Application UX* report — flat design + improperly rescaled layouts
  are the top usability threats; users rely only on basic gestures (tap, press,
  swipe, drag, pinch). `nngroup.com/articles/tablet-usability/`
- **UX for the Masses — Tablet-friendly case study** — practical tablet layout/
  touch lessons. `uxforthemasses.com`

## Engineering / architecture
- **react-made-native-easy** — RN project structure, container/presentational
  separation, Redux, styling and testing conventions.
  `react-made-native-easy.github.io`
- **Non-functional requirements** (ScienceDirect topic) — standard NFR
  categories used as the release-gate lens in
  [`03-optimization/optimization-categories.md`](03-optimization/optimization-categories.md).
- Inline article material provided in the task brief: React/RN architecture
  (S. Hudge), React Native CLI kiosk mode / Device Owner + Lock Task (T. Noman),
  tablet-first TypeScript + responsive design (D. Şahin).

## Open-source reference apps (documented in `repo-examples/`)
- **fbsamples/f8app** — official F8 2017 app (React Native, Redux, Relay/GraphQL,
  Parse Server). `github.com/fbsamples/f8app` · tutorials `makeitopen.com`.
- **React Native UIExplorer → RNTester** — official component gallery.
  Classic mirror `github.com/rnplay/uiexplorer`; current `facebook/react-native`
  at `packages/rn-tester`.
- **taskrabbit/ReactNativeSampleApp** — early architecture-focused sample (Flux,
  isolated network layer, URL routing). `github.com/taskrabbit/ReactNativeSampleApp`.

> Snippets in `repo-examples/` were fetched from `raw.githubusercontent.com` and
> cited with paths/permalinks. The GitHub tree/contents API was rate-limited (403)
> during research, so some directory listings came from GitHub web tree views;
> each doc flags what it could not fully verify. These repos are dated — study the
> patterns, not the verbatim APIs.

## Provisional architecture decision
The app stack is **not finalized**. Working assumption (per project owner):
**React Native + TypeScript for UI/UX**, **Kotlin/Java native modules for device
support** (kiosk Device Owner / Lock Task, low-latency input, some peripherals).
Docs keep the native↔JS boundary explicit and avoid over-committing. Revisit if
the architecture is decided.

> Retrieval note: Android design/quality/large-screen pages were fetched
> directly. NN/g, the PDFs, ScienceDirect, and the GitHub Pages guide return 403
> to automated fetchers; their specifics here were taken from search-result
> extracts and the original article text in the task brief. Verify exact
> wording against the primary URLs if quoting externally.
