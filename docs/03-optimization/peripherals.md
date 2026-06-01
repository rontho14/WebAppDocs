# Peripheral Support

Kiosks commonly attach hardware: barcode/QR scanners, receipt printers, card
readers, NFC, external keyboards. Plan integration explicitly.

## Common peripherals & approach
| Peripheral | Typical interface | RN approach |
|-----------|-------------------|-------------|
| Barcode/QR scanner | USB/BT HID (acts as keyboard) | Capture keystrokes; or camera scan lib |
| Receipt/label printer | USB / BT / network (ESC-POS) | Native module or vendor SDK bridge |
| Card reader / POS | USB / BT SDK | Vendor SDK via native module |
| NFC | Built-in | `react-native-nfc-manager` |
| External keyboard | USB / BT | Handle key + focus events |

## HID scanner (keyboard-wedge) capture
Most scanners "type" the code then send Enter. Capture into a hidden/auto-focused input:
```tsx
const ref = useRef<TextInput>(null);
useEffect(() => { ref.current?.focus(); }, []); // keep focus to catch scans
<TextInput
  ref={ref}
  showSoftInputOnFocus={false}     // suppress on-screen keyboard
  onSubmitEditing={(e) => onScan(e.nativeEvent.text)} // Enter = scan complete
  blurOnSubmit={false}
/>
```

## Camera-based scanning
Use `react-native-vision-camera` + code scanner when no hardware scanner exists.

## Native module bridge (printers / SDKs)
Wrap the vendor SDK in a native module and expose a clean JS API (same pattern as
the kiosk module — see [`../04-kiosk/lock-task-mode.md`](../04-kiosk/lock-task-mode.md)):
```ts
import { NativeModules } from 'react-native';
const { ReceiptPrinter } = NativeModules;
export const printReceipt = (payload: string) => ReceiptPrinter.print(payload);
```

## Rules
- Detect connect/disconnect; show clear status and recover gracefully.
- Request USB/BT permissions; whitelist USB devices for kiosk auto-grant.
- Keep peripheral logic in `services/` behind a typed interface so screens stay clean.
- Test with the *exact* hardware model — vendor SDKs vary widely.
