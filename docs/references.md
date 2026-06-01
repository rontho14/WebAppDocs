# References

Primary sources behind this knowledge base. Citations in the docs point here.

## Android (official)
- **Adaptive / Large-Screen App Quality guidelines** — the Tier 1/2/3 checklist,
  config & continuity, multi-window, input, and the test-device matrix.
  `developer.android.com/docs/quality-guidelines/.../large-screen-app-quality`
- **Large screens guide** — window size classes (Compact/Medium/Expanded) and
  canonical layouts (List-Detail, Feed, Supporting Pane).
  `developer.android.com/guide/topics/large-screens`
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

> Retrieval note: Android design/quality/large-screen pages were fetched
> directly. NN/g, the PDFs, ScienceDirect, and the GitHub Pages guide return 403
> to automated fetchers; their specifics here were taken from search-result
> extracts and the original article text in the task brief. Verify exact
> wording against the primary URLs if quoting externally.
