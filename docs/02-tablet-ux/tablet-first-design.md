# Tablet-First Design

Design for the tablet (large screen, landscape) from day one, then adapt down.
Don't stretch a phone layout — use the extra space.

> ⚠️ NN/g's tablet research: the **top two threats to tablet usability are flat
> design** (users can't tell what's tappable) **and improperly rescaled phone
> layouts**. Avoid both. Source: [`../references.md`](../references.md).

## Detect the device
```ts
// utils/responsive.ts
import { Dimensions, PixelRatio } from 'react-native';
const { width, height } = Dimensions.get('window');

export const deviceInfo = {
  width, height,
  isTablet: width >= 768,
  isPhone: width < 768,
  orientation: width > height ? 'landscape' : 'portrait',
};

export const responsive = {
  wp: (pct: number) => Math.round(PixelRatio.roundToNearestPixel((pct * width) / 100)),
  fontSize: (size: number) => Math.round(size * (width / 375)), // 375 = base phone width
  gridColumns: deviceInfo.isTablet ? 2 : 1,
};
```
> ⚠️ `Dimensions.get` is captured once. For split-screen/rotation use the
> `useWindowDimensions()` hook so values update. See
> [`../03-optimization/multi-window-continuity.md`](../03-optimization/multi-window-continuity.md).

## Breakpoints — Android Window Size Classes (official)
Classes are based on **available window space** (not physical size) and are
**dynamic** — they change with rotation, multi-window, and folding. **Width is
usually the deciding axis** (content scrolls vertically). Source:
[`../references.md`](../references.md).

**Width:**
| Width (dp) | Class | Typical device |
|-----------|-------|----------------|
| `< 600` | **Compact** | phones in portrait |
| `600–839` | **Medium** | tablets in portrait, unfolded inner displays (portrait) |
| `840–1199` | **Expanded** | tablets in landscape, unfolded (landscape) |
| `1200–1599` | **Large** | large tablet displays |
| `≥ 1600` | **Extra-large** | desktop displays |

**Height:**
| Height (dp) | Class |
|-----------|-------|
| `< 480` | **Compact** (phones landscape) |
| `480–899` | **Medium** |
| `≥ 900` | **Expanded** (tablets portrait) |

**Layout decisions:** Compact width → single pane, stacked. Medium → transitional /
two-pane. Expanded+ → multi-pane (list-detail, supporting pane). **Compact height
→ avoid two-pane even at medium width** (e.g. phone in landscape).

## Canonical layouts (pick one per screen)
Android recommends three proven adaptive patterns — implement the matching one.
Pane proportions below are the official defaults:

- **List-Detail** — list + detail. **Expanded:** both panes side-by-side, selecting
  an item updates detail. **Medium/Compact:** one pane at a time; selection pushes
  detail, back returns to list. The primary kiosk/tablet pattern (see two-pane below).
- **Feed** — adaptive grid of cards; column count grows with width (set a minimum
  column width, e.g. ~180 dp; single column on Compact).
- **Supporting Pane** — primary content + secondary pane that is only meaningful
  *with* the primary. **Expanded ≈ 70/30**, **Medium ≈ 50/50**, **Compact:** primary
  only (supporting content hidden or in a bottom sheet).

## Two-pane (master-detail) — the key tablet pattern
```tsx
export function VisitorsScreen() {
  const { width } = useWindowDimensions();
  const twoPane = width >= 840;
  return (
    <View style={{ flex: 1, flexDirection: 'row' }}>
      <VisitorList style={{ flex: twoPane ? 1 : 1 }} />
      {twoPane && <VisitorDetail style={{ flex: 2 }} />}
    </View>
  );
}
```
On phones the detail becomes a pushed screen; on tablets it sits beside the list.

## Responsive card grid
```tsx
<FlatList
  data={tasks}
  numColumns={deviceInfo.isTablet ? 2 : 1}
  key={deviceInfo.isTablet ? 'tablet' : 'phone'} // remount on column change
  renderItem={({ item }) => <TaskCard task={item} />}
/>
```

## Debug current device while testing
```ts
console.log('Device:', {
  width: deviceInfo.width, height: deviceInfo.height,
  isTablet: deviceInfo.isTablet, columns: responsive.gridColumns,
});
```

## Rules
- Use flex + percentages, never fixed pixel widths for containers.
- Scale type/spacing through tokens ([`../01-architecture/theme-design-tokens.md`](../01-architecture/theme-design-tokens.md)).
- Cap content max-width on huge screens so lines stay readable.
- Test phone, 7" and 10" tablets, both orientations (matrix in [`../03-optimization/android-quality-checklist.md`](../03-optimization/android-quality-checklist.md)).
