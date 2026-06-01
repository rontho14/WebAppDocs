# Input Device Support (keyboard, mouse, trackpad, stylus)

Human input devices — distinct from [`peripherals.md`](peripherals.md) (scanners,
printers, card readers). Tablets/kiosks are often used with an external keyboard,
mouse, or stylus. These are **Tier 2/1** criteria in the Android quality
guidelines ([`android-quality-checklist.md`](android-quality-checklist.md)).

## External keyboard
- **Focus order:** logical Tab / arrow navigation through interactive elements.
  Chain inputs with refs (`returnKeyType="next"`, focus the next field).
- **Shortcuts:** support cut/copy/paste/undo/redo; **Enter = submit/send**;
  **Space = play/pause** for media.
- Switch between physical and on-screen keyboard **without relaunching**.
```tsx
// react-native: capture hardware key events
<View onKeyDown={(e) => { if (e.nativeEvent.key === 'Enter') submit(); }}>
// or per-input:
<TextInput onSubmitEditing={submit} blurOnSubmit={false} returnKeyType="send" />
```
> Full hardware-key handling (Tab traversal, global shortcuts) may need a small
> **native module** that forwards `KeyEvent`s — flag for native if RN props fall short.

## Mouse & trackpad
- Click every actionable element; select radio/checkbox/text; scroll both axes.
- **Right-click / secondary tap** → context menu (appears next to the item).
- **Hover states** on actionable elements (`onMouseEnter`/`onMouseLeave` on web/RN-Web;
  `Pressable` style callbacks on RN where supported).
- **Zoom:** Ctrl+scroll (mouse) / pinch (trackpad) where zoom applies.
- Show a **scrollbar while scrolling** with a pointer.

## Stylus
- **Baseline (Tier 3):** select/manipulate UI, scroll lists & pickers — works by default.
- **Tier 1 (drawing apps):** draw/erase, pressure-varying stroke width, tilt shading,
  **palm & finger rejection**, low-latency + motion prediction.
- Stylus drawing/erasing and low-latency ink are **native** concerns — use a
  dedicated drawing lib or native canvas; don't attempt in pure RN.

## Focus & accessibility (applies to all input)
- Provide visible focus indicators when **not** in touch mode.
- Set `accessibilityLabel` / `accessibilityRole` on every interactive element.
- Maintain a sensible `accessibilityViewIsModal` for dialogs.

## Kiosk reality check
A locked kiosk usually exposes **no keyboard/mouse to the public**, but admin/
setup flows and attached scanners still generate key events. Cover:
- On-screen keyboard for visitor input (see [`../02-tablet-ux/forms-inputs.md`](../02-tablet-ux/forms-inputs.md)).
- Hardware key events from scanners (see [`peripherals.md`](peripherals.md)).
- Keyboard nav for the **admin** screens behind the PIN.

## RN vs native summary
| Need | Where |
|------|-------|
| Tab/arrow focus, Enter submit, hover, right-click menu | RN/TS |
| Global hardware shortcuts, raw `KeyEvent` capture | native module |
| Stylus ink (pressure/tilt/palm rejection) | native / drawing lib |
