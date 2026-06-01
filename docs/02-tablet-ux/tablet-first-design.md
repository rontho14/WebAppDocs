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
| Width (dp) | Class | Layout |
|-----------|-------|--------|
| `< 600` | **Compact** (phone) | single column, stacked |
| `600–839` | **Medium** (small tablet, unfolded) | 2 columns / master-detail collapsible |
| `≥ 840` | **Expanded** (large tablet, desktop) | two-pane (list + detail), persistent nav rail |

## Canonical layouts (pick one per screen)
Android recommends three proven adaptive patterns — implement the matching one:
- **List-Detail** — list + detail side-by-side on Expanded; the primary kiosk/tablet pattern (see two-pane below).
- **Feed** — resizable grid of cards (`numColumns` scales with width).
- **Supporting Pane** — main content + a secondary tools/info pane on wide screens.

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

## Rules
- Use flex + percentages, never fixed pixel widths for containers.
- Scale type/spacing through tokens ([`../05-quality`](../05-quality/typescript-strict.md) theme).
- Cap content max-width on huge screens so lines stay readable.
- Test phone, 7" and 10" tablets, both orientations.
