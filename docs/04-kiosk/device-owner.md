# Kiosk: Device Owner (the real prerequisite)

`activity.startLockTask()` alone is **fake** kiosk mode — Android shows
confirmation dialogs and allows escape. Real lockdown requires the app to be the
**Device Owner**. Then Lock Task runs silently with full control.

> **Stack requirement:** React Native **CLI**, not Expo. Device Owner needs
> native Android APIs Expo doesn't expose.

## What Device Owner unlocks
- Silent kiosk start (no dialogs)
- App whitelisting for Lock Task
- Full system lockdown (status bar, home, recents)
- Policy control (disable factory reset, USB, etc.)

## Provisioning requirements
The device must be **freshly factory-reset**: no Google accounts, no other users.
Install the app first, then set Device Owner via adb:
```bash
adb shell dpm set-device-owner com.yourapp/.KioskDeviceAdminReceiver
```
> `Invalid component / already provisioned` → device already has an owner or an
> account. **Factory reset** and retry. For fleets, provision via QR/NFC/EMM enrollment.

## 1. DeviceAdminReceiver (can be empty)
`android/app/src/main/java/com/yourapp/KioskDeviceAdminReceiver.java`
```java
package com.yourapp;
import android.app.admin.DeviceAdminReceiver;
public class KioskDeviceAdminReceiver extends DeviceAdminReceiver { }
```

## 2. Policy metadata — `res/xml/device_admin.xml`
```xml
<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
  <uses-policies>
    <force-lock />
    <wipe-data />
  </uses-policies>
</device-admin>
```

## 3. AndroidManifest — register the receiver
```xml
<receiver
  android:name=".KioskDeviceAdminReceiver"
  android:permission="android.permission.BIND_DEVICE_ADMIN"
  android:exported="true">          <!-- REQUIRED, or owner setup fails silently -->
  <meta-data
    android:name="android.app.device_admin"
    android:resource="@xml/device_admin" />
  <intent-filter>
    <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
  </intent-filter>
</receiver>
```

## Verify
```bash
adb shell dumpsys device_policy | grep -i "device owner"
```

➡️ Once Device Owner is set, wire Lock Task: [`lock-task-mode.md`](lock-task-mode.md).
➡️ Then harden escape routes: [`security-config.md`](security-config.md).
