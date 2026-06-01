# Android Large-Screen App Quality — Full Checklist (reference)

> **When to read:** auditing the app against Google's large-screen quality bar.
> This is a long reference — load it only for a quality pass, not every run.
> Criteria reproduced from Android's Large-Screen App Quality guidelines.
> Source: [`../references.md`](../references.md).

Three tiers. **Target Tier 2 minimum; Tier 1 for premium kiosks.** IDs (LS-*, T#-#)
mirror the official guideline so you can cross-check. "Native?" flags whether an
item is pure RN/TS UI or likely needs a native module.

---

## Tier 3 — Large Screen Ready (baseline)
Critical flows complete; app runs full-screen, **no letterboxing / compatibility mode**.

### Configuration & continuity (LS-C1, LS-C2)
| ID | Criterion | Native? |
|----|-----------|---------|
| T3-1 | Not letterboxed / not in compat mode in portrait, landscape, multi-window, or unfolded. Resizing keeps correct orientation & state. | manifest (`resizeableActivity`, `configChanges`) |
| T3-2 | After rotate / fold / unfold / resize: **scroll position kept, playback resumes, entered text retained**. | RN/TS |
| T3-3 | Handles **combined** config changes (rotate + resize + fold) without crash/state loss. | RN/TS |

### Multi-window & multi-resume (LS-M1, LS-M2)
| ID | Criterion | Native? |
|----|-----------|---------|
| T3-4 | Fully functional in multi-window at all sizes/orientations/foldable states. | RN/TS + manifest |
| T3-5 | UI keeps updating when **not top-focused** (video, downloads, messages). | RN/TS |
| T3-6 | Releases exclusive resources (camera, mic) in multi-window; regains on refocus. | native/module |

### Camera preview & media projection (LS-CM1, LS-CM2)
| ID | Criterion | Native? |
|----|-----------|---------|
| T3-7 | Camera preview correct (orientation/proportions) in all states & multi-window. | native/lib |
| T3-8 | Media projection correct in all states & multi-window. | native/lib |

### Keyboard, mouse & trackpad (LS-I1, LS-I2)
| ID | Criterion | Native? |
|----|-----------|---------|
| T3-9 | External keyboard works; switches physical⇄virtual without relaunch. | RN/TS |
| T3-10 | Mouse/trackpad: click all elements, select radio/checkbox/text, scroll both axes. | RN/TS |

### Stylus (LS-S1)
| ID | Criterion | Native? |
|----|-----------|---------|
| T3-11 | Basic stylus: select/manipulate elements, scroll lists & pickers (default). | RN/TS |
| T3-12 | Android 14+: text entry in fields w/o soft keyboard. ChromeOS M114+: WebView input. | native |

---

## Tier 2 — Large Screen Optimized (project target)
Layout optimized for every size + richer input.

### UX & layout (LS-U1–U4)
| ID | Criterion | Native? |
|----|-----------|---------|
| T2-1 | **Responsive + adaptive** layouts via window size classes: nav rail expands to panel on large screens; grids scale columns; trailing panels open by default on desktop, closed on small. | RN/TS |
| T2-1b | Modals/menus sized right: bottom sheets **max-width** (not full); buttons **not full-width**; text fields **not stretched**; edit menus don't cover whole screen; **context menus next to the selected item**; **nav rail replaces bottom bar**; images proper resolution (not stretched/cropped). | RN/TS |
| T2-2 | **Touch targets ≥ 48 dp** at all display sizes. | RN/TS |
| T2-3 | Custom drawables focusable via keyboard/D-pad with visible focus when not in touch mode. | RN/TS |

### Keyboard, mouse & trackpad (LS-I3–I9)
| ID | Criterion | Native? |
|----|-----------|---------|
| T2-4 | Main flows support keyboard nav: **Tab + arrow keys**. | RN/TS |
| T2-5 | Shortcuts: select, cut, copy, paste, undo, redo. | RN/TS |
| T2-6 | Media: **Spacebar** plays/pauses. | RN/TS |
| T2-7 | **Enter = send** in communication/entry flows. | RN/TS |
| T2-8 | Right-click / secondary tap opens options menu. | RN/TS |
| T2-9 | Zoom via **Ctrl+scroll** (mouse) or **pinch** (trackpad). | RN/TS |
| T2-10 | Hover states on actionable elements. | RN/TS |

See [`input-support.md`](input-support.md) for how to implement these in RN.

---

## Tier 1 — Large Screen Differentiated (premium)
| ID | Criterion | Native? |
|----|-----------|---------|
| T1-1 | Picture-in-picture: enter/exit & resize across all states. | native |
| T1-2 | Launch another app side-by-side (`FLAG_ACTIVITY_LAUNCH_ADJACENT`). | native |
| T1-3 | Open/close attachments & notifications across all states. | RN/TS |
| T1-4 | Multiple app instances in separate windows. | native |
| T1-5 | Foldable postures: **tabletop** (calls/playback), **book** (reading), **dual-display**. | native |
| T1-6 | Camera adjusts preview folded/unfolded; front/back preview. | native |
| T1-7 | Drag & drop within app and across apps (touch/mouse/trackpad/stylus). | native + RN |
| T1-8 | Comprehensive keyboard shortcuts; parity with web/desktop (Ctrl-C/Z…). | RN/TS |
| T1-9 | Modifier combos: Ctrl/Shift+click range & multi-select. | RN/TS |
| T1-10 | Scrollbar shown while scrolling with mouse/trackpad. | RN/TS |
| T1-11 | Hover reveals fly-out menus / tooltips. | RN/TS |
| T1-12 | Desktop-style menus & context menus. | RN/TS |
| T1-13 | Reconfigurable/resizable panels in multi-pane layouts (not nav bars/rails). | RN/TS |
| T1-14 | Triple-click/tap selects line/paragraph. | RN/TS |
| T1-15 | Stylus draw/write + erase. | native/lib |
| T1-16 | Stylus drag & drop within/across apps. | native |
| T1-17 | Enhanced stylus: low latency, pressure, tilt, palm/finger rejection. | native |
| T1-18 | Custom cursors signal mode (I-beam, crosshair, magnifier, tool). | native |

---

## Test-device matrix (minimum)
| Device | Resolution (dp) |
|--------|-----------------|
| Foldable (unfolded) | 841 × 701 |
| 8" tablet | 1024 × 640 |
| 10.5" tablet | 1280 × 800 |
| 13" Chromebook | 1600 × 900 |

Emulators: 7.6" fold-in foldable, Pixel C 9.94" tablet, Surface Duo (dual-display).
Test each in **portrait + landscape** and in **split-screen**.

> Kiosk note: many Tier-1 items (multi-instance, PiP, foldable postures) are
> irrelevant to a single-purpose locked kiosk. Prioritize Tier 3 continuity +
> Tier 2 layout/input; treat Tier 1 as optional per hardware.
