# Multi-Window & Continuity

Tablets resize: rotation, split-screen, free-form windows. The app must adapt
live and never lose user state.

## React to size changes at runtime
Use the hook (reactive), not `Dimensions.get` (one-shot):
```tsx
import { useWindowDimensions } from 'react-native';

function Screen() {
  const { width, height } = useWindowDimensions(); // updates on resize/rotate
  const landscape = width > height;
  // recompute layout from these values
}
```

## Declare resize support (Android)
`AndroidManifest.xml`:
```xml
<activity
  android:name=".MainActivity"
  android:resizeableActivity="true"
  android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|screenSize|smallestScreenSize|uiMode"
  ... />
```
`configChanges` lets the activity handle resize/rotate **without recreating** —
RN re-renders via `useWindowDimensions` instead of remounting.

## What Android quality testing actually checks (continuity)
After rotation, fold/unfold, **and** window resize, the app must retain:
- **Scroll position** of scrollable content,
- **Playback position** of media,
- **Text already entered** in fields,
and survive **combined** changes (rotate + resize + fold) without crashing.
It must also keep updating UI when **not top-focused** in multi-window, and
release exclusive resources (camera/mic) when backgrounded, regaining them on
refocus. Source: [`../references.md`](../references.md).

## Preserve state across config changes
- Keep transient form state in component state; it survives re-render (no remount).
- Keep important state in the store ([`../01-architecture/state-management.md`](../01-architecture/state-management.md)).
- For process death, persist critical state (`redux-persist` / AsyncStorage) and rehydrate on launch.

## Patterns that adapt
- Two-pane ⇄ single-pane based on width ([`../02-tablet-ux/tablet-first-design.md`](../02-tablet-ux/tablet-first-design.md)).
- `FlatList numColumns` recomputed from width (remount via `key`).
- Avoid absolute positioning tied to a fixed width/height.

## Kiosk note
Even when locked to one app, allow rotation only if the hardware mount needs it;
otherwise pin orientation (`android:screenOrientation="sensorLandscape"` or
`"locked"`) for a stable single-purpose UI.
