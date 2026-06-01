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

### Tier 3 — Large Screen Ready (baseline)
Critical flows complete; app runs full-screen, **no letterboxing / compatibility mode**.
- [ ] **Config & continuity:** not letterboxed in portrait/landscape, multi-window, or unfolded. On rotation/resize/fold: **scroll position, playback position, and entered text are retained**.
- [ ] Handles **combined** config changes (rotate + resize + fold) without crash or state loss.
- [ ] **Multi-window:** fully functional at all window sizes; keeps updating UI when not top-focused (downloads, video); releases camera/mic when backgrounded, regains on refocus.
- [ ] **Input:** external keyboard works; switches physical⇄virtual keyboard without relaunch. Mouse/trackpad can click, select, scroll. Basic stylus select/scroll.

### Tier 2 — Large Screen Optimized (target for this project)
Layout optimized for every size + richer input.
- [ ] **Responsive + adaptive layouts** via window size classes (see breakpoints in [`../02-tablet-ux/tablet-first-design.md`](../02-tablet-ux/tablet-first-design.md)).
- [ ] **Touch targets ≥ 48 dp** on all display sizes ([`../02-tablet-ux/touch-targets.md`](../02-tablet-ux/touch-targets.md)).
- [ ] Modals/menus sized for large screens: bottom sheets/buttons/text fields **not full width**; context menus appear **next to the selected item**; **navigation rail replaces bottom bar** on large screens.
- [ ] Keyboard navigation on main flows (Tab + arrows); shortcuts (cut/copy/paste/undo/redo); right-click context menus; hover states; Ctrl+scroll / pinch zoom.

### Tier 1 — Large Screen Differentiated (premium)
- [ ] Multi-instance / multi-window launch, picture-in-picture, drag & drop (touch+mouse+stylus), foldable postures, enhanced stylus (pressure/tilt/palm rejection), custom cursors.

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
