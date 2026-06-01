# Kiosk: Lock Task Mode (native bridge)

Prerequisite: app is **Device Owner** ([`device-owner.md`](device-owner.md)).
This wires a native module to start/stop Lock Task and exposes a clean JS API.

## 1. Native module — `KioskLockTaskModule.java`
```java
public class KioskLockTaskModule extends ReactContextBaseJavaModule {
  public KioskLockTaskModule(ReactApplicationContext c) { super(c); }

  @Override public String getName() { return "KioskLockTask"; }

  @ReactMethod public void startLockTask() {
    Activity activity = getCurrentActivity();
    if (activity == null) return;

    DevicePolicyManager dpm = (DevicePolicyManager)
        getReactApplicationContext().getSystemService(Context.DEVICE_POLICY_SERVICE);
    ComponentName admin = new ComponentName(
        getReactApplicationContext(), KioskDeviceAdminReceiver.class);

    String pkg = getReactApplicationContext().getPackageName();
    if (dpm.isDeviceOwnerApp(pkg)) {
      dpm.setLockTaskPackages(admin, new String[]{ pkg }); // whitelist self
    }
    activity.startLockTask();
  }

  @ReactMethod public void stopLockTask() {
    Activity activity = getCurrentActivity();
    if (activity != null) activity.stopLockTask();
  }
}
```

## 2. React package — `KioskLockTaskPackage.java`
```java
public class KioskLockTaskPackage implements ReactPackage {
  @Override public List<NativeModule> createNativeModules(ReactApplicationContext c) {
    return Arrays.asList(new KioskLockTaskModule(c));
  }
  @Override public List<ViewManager> createViewManagers(ReactApplicationContext c) {
    return Collections.emptyList();
  }
}
```
Register in `MainApplication`: `packages.add(new KioskLockTaskPackage());`

## 3. Manifest — Lock Task launcher
```xml
<activity android:name=".MainActivity" android:exported="true"
          android:lockTaskMode="if_whitelisted">
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
    <category android:name="android.intent.category.HOME" /> <!-- become default launcher -->
  </intent-filter>
</activity>
```

## 4. JS wrapper — `kiosk.ts`
```ts
import { NativeModules, Platform } from 'react-native';
const { KioskLockTask } = NativeModules;

export const startKioskMode = () => { if (Platform.OS === 'android') KioskLockTask.startLockTask(); };
export const stopKioskMode  = () => { if (Platform.OS === 'android') KioskLockTask.stopLockTask(); };
```

## 5. Auto-start on launch
```tsx
useEffect(() => {
  const t = setTimeout(() => startKioskMode(), 1500); // let UI mount first
  return () => clearTimeout(t);
}, []);
```

## 6. Exit (always provide one)
Expose `stopKioskMode()` behind a protected admin path — never a visible button.
See PIN/gesture pattern in [`security-config.md`](security-config.md).

## Troubleshooting
- **Lock task doesn't start** → wrong package name, Device Owner not set, or receiver misregistered.
- **Dialogs still appear** → app is not Device Owner (re-provision).
