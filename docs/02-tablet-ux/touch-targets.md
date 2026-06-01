# Touch Targets

Kiosks are touched by strangers, sometimes standing, sometimes gloved. Be generous.

## Minimum sizes
| Source | Minimum |
|--------|---------|
| Android (Material) / large-screen quality | **48 × 48 dp** |
| NN/g (research-based, adult fingertip) | **~1 cm × 1 cm** (≈ 10 mm physical) |
| Kiosk recommendation | **56–64 dp** for primary actions |
| Spacing between targets | **≥ 8 dp** |

> NN/g's "Touch Targets on Touchscreens" finds the average adult fingertip is
> ~1 cm wide, so the *physical* target should be ~1 cm² regardless of pixel
> density — 48 dp is the digital floor, not a ceiling. Source:
> [`../references.md`](../references.md).

## Enforce a minimum
```ts
// theme/touch.ts
export const TOUCH = { min: 48, comfortable: 56, primary: 64, gap: 8 };
```
```tsx
<Pressable
  style={{ minWidth: TOUCH.comfortable, minHeight: TOUCH.comfortable,
           justifyContent: 'center', alignItems: 'center' }}
  hitSlop={8}            // extend tappable area beyond visual bounds
  android_ripple={{ color: '#ddd' }}>
  <Text>Check in</Text>
</Pressable>
```

## Rules
- Visual size can be small, but pad/`hitSlop` to reach the minimum.
- Primary kiosk actions (Start, Confirm) should be large and high-contrast.
- Keep ≥ 8 dp between adjacent targets to prevent mis-taps.
- Give immediate feedback on press (ripple, scale, color) — no "dead" taps.
- Place primary actions in the thumb-reachable lower/side zones for handheld tablets.
- Don't rely on hover or long-press as the *only* way to reach an action.
- Disable text selection on labels/buttons in kiosk mode (`selectable={false}`).
