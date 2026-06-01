# Knowledge Base — Mobile Kiosk Apps (React Native, Tablet-First)

> **For the AI agent:** This is a routing index. **Do not load every file.**
> Read this table, then open **only** the document(s) relevant to the current
> task. Each doc is self-contained and kept short to protect your context window.

## How to use this knowledge base

1. Identify the task domain (architecture? UX? kiosk lockdown? testing?).
2. Open the matching file from the table below.
3. Load a second file only if the task genuinely spans two domains.

## Index

### 01 — Architecture (`01-architecture/`)
| File | Read it when you need to… |
|------|---------------------------|
| [`directory-structure.md`](01-architecture/directory-structure.md) | Decide where a file/folder goes; scaffold a feature |
| [`architecture-patterns.md`](01-architecture/architecture-patterns.md) | Pick a pattern (container/presentational, atomic, feature-based) |
| [`state-management.md`](01-architecture/state-management.md) | Choose local vs global state; wire Redux Toolkit / Context / Query |
| [`service-layer.md`](01-architecture/service-layer.md) | Add an API call; structure `services/`; configure Axios + caching |

### 02 — Tablet UX (`02-tablet-ux/`)
| File | Read it when you need to… |
|------|---------------------------|
| [`tablet-first-design.md`](02-tablet-ux/tablet-first-design.md) | Build responsive layouts, breakpoints, two-pane patterns |
| [`touch-targets.md`](02-tablet-ux/touch-targets.md) | Size/space tappable elements; accessibility minimums |
| [`navigation-menus.md`](02-tablet-ux/navigation-menus.md) | Choose stack/tab/drawer/rail; tablet navigation patterns |
| [`forms-inputs.md`](02-tablet-ux/forms-inputs.md) | Build inputs, keyboards, validation for touch kiosks |

### 03 — Optimization & Devices (`03-optimization/`)
| File | Read it when you need to… |
|------|---------------------------|
| [`optimization-categories.md`](03-optimization/optimization-categories.md) | Apply the Android large-screen quality checklist |
| [`multi-window-continuity.md`](03-optimization/multi-window-continuity.md) | Handle resize, split-screen, rotation, state restoration |
| [`peripherals.md`](03-optimization/peripherals.md) | Support keyboards, scanners, printers, card readers, USB/BLE |

### 04 — Kiosk Lockdown (`04-kiosk/`)
| File | Read it when you need to… |
|------|---------------------------|
| [`device-owner.md`](04-kiosk/device-owner.md) | Provision Device Owner; the native module + receiver setup |
| [`lock-task-mode.md`](04-kiosk/lock-task-mode.md) | Start/stop Lock Task; whitelist apps; JS wrapper + auto-start |
| [`security-config.md`](04-kiosk/security-config.md) | Block escape routes, secure storage, disable system UI |

### 05 — Quality (`05-quality/`)
| File | Read it when you need to… |
|------|---------------------------|
| [`typescript-strict.md`](05-quality/typescript-strict.md) | Configure strict TS, path aliases, shared types |
| [`testing-tdd.md`](05-quality/testing-tdd.md) | Write unit/E2E tests; follow the TDD loop; mock APIs |
| [`error-handling-performance.md`](05-quality/error-handling-performance.md) | Add error boundaries, logging, memoization, lazy loading |

### Sources
[`references.md`](references.md) — primary sources (Android quality guidelines,
NN/g tablet UX, etc.) behind every cited claim.

## Global conventions (apply everywhere)

- **Stack:** React Native CLI (not Expo — Device Owner needs native APIs), TypeScript strict.
- **Target:** Android tablets in kiosk/single-purpose mode.
- **Style:** Functional components + hooks only. No class components except `ErrorBoundary`.
- **Imports:** Use the `@/` path alias (see `typescript-strict.md`).
- **Keep docs enxutos:** short, code-first, no filler.
