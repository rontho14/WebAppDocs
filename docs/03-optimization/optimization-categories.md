# Optimization Categories (Android Large-Screen App Quality)

Based on Google's **Adaptive / Large-Screen App Quality** guidelines and the
2022 Play Store changes that **rank and feature apps by large-screen quality**
(accounting for orientation support, keyboard support, etc.) and show
**device-specific ratings/reviews**. Optimizing here directly affects
discoverability, not just polish.

> Sources: Android large-screen app quality guidelines; "Helping Users Discover
> Quality Apps on Large Screens" (Android Developers Blog, 2022). See
> [`../references.md`](../references.md).

## The three tiers (target Tier 2 minimum; Tier 1 for premium kiosks)

- **Tier 3 — Ready:** full-screen, no letterboxing; survives rotate/resize/fold
  with scroll/playback/text retained; works in multi-window; basic keyboard/
  mouse/stylus.
- **Tier 2 — Optimized (project target):** responsive + adaptive layouts via
  window size classes; touch targets ≥ 48 dp; modals/menus sized for large
  screens (nav rail replaces bottom bar; context menu next to item); keyboard
  nav + shortcuts + hover + zoom.
- **Tier 1 — Differentiated (premium):** multi-instance, PiP, drag & drop,
  foldable postures, enhanced stylus, custom cursors.

➡️ **Full testable criteria (all LS-* / T#-# items + native-vs-RN notes):
[`android-quality-checklist.md`](android-quality-checklist.md).**
➡️ Input-device detail: [`input-support.md`](input-support.md).

## Canonical layouts (Android)
Prefer one of: **List-Detail** (master-detail — primary tablet pattern),
**Feed**, **Supporting Pane**. See [`../02-tablet-ux/tablet-first-design.md`](../02-tablet-ux/tablet-first-design.md).

## Non-functional requirements lens (use as a release gate)
Beyond the tiers, evaluate standard NFRs for an unattended kiosk:
**performance, reliability, availability, usability, security, scalability,
maintainability, portability**. Each maps to a doc:
- Performance/reliability → [`../05-quality/error-handling-performance.md`](../05-quality/error-handling-performance.md)
- Usability → `../02-tablet-ux/`
- Security → [`../04-kiosk/security-config.md`](../04-kiosk/security-config.md)
- Maintainability → `../01-architecture/` + [`../05-quality/typescript-strict.md`](../05-quality/typescript-strict.md)

## Test device matrix (from Android guidelines)
| Device | Resolution (dp) |
|--------|-----------------|
| Foldable (unfolded) | 841 × 701 |
| 8" tablet | 1024 × 640 |
| 10.5" tablet | 1280 × 800 |
| 13" Chromebook | 1600 × 900 |

Test each in portrait + landscape, and in split-screen.
