# Kiosk: Security Configuration

Locking the task is not enough — block every escape route, protect data, and
control the exit. Requires Device Owner ([`device-owner.md`](device-owner.md)).

## Block escape routes (DevicePolicyManager, Device Owner only)
```java
// disable status bar pull-down (notifications, quick settings)
dpm.setStatusBarDisabled(admin, true);

// keep screen on while plugged in
dpm.setGlobalSetting(admin, Settings.Global.STAY_ON_WHILE_PLUGGED_IN, "3");

// prevent factory reset / safe boot / config changes
dpm.addUserRestriction(admin, UserManager.DISALLOW_FACTORY_RESET);
dpm.addUserRestriction(admin, UserManager.DISALLOW_SAFE_BOOT);
dpm.addUserRestriction(admin, UserManager.DISALLOW_ADD_USER);
dpm.addUserRestriction(admin, UserManager.DISALLOW_MOUNT_PHYSICAL_MEDIA);

// suppress system error dialogs (ANR/crash) in kiosk
dpm.setGlobalSetting(admin, "hide_error_dialogs", "1");
```

## Lock Task feature flags (what stays available while locked)
```java
dpm.setLockTaskFeatures(admin,
    DevicePolicyManager.LOCK_TASK_FEATURE_HOME |       // home button -> your app
    DevicePolicyManager.LOCK_TASK_FEATURE_GLOBAL_ACTIONS); // long-press power
// omit SYSTEM_INFO / NOTIFICATIONS to hide them
```

## Auto-recover into kiosk
- Re-enter Lock Task on app resume and after reboot.
- Listen for `BOOT_COMPLETED` (Device Owner can auto-launch) to relaunch the app.
- Wrap the app in an error boundary that restarts the kiosk flow, not exits it.

## Protected exit (the ONLY way out)
Never expose a visible exit. Use a hidden gesture + PIN:
```tsx
// e.g. 5 taps on the logo within 3s -> PIN modal -> stopKioskMode()
function useAdminExit() {
  const taps = useRef(0);
  const onSecretTap = () => {
    if (++taps.current >= 5) { taps.current = 0; promptPin(); }
  };
  const promptPin = () => { /* compare against env.KIOSK_EXIT_PIN, then stopKioskMode() */ };
  return { onSecretTap };
}
```
- Store the PIN/secrets in env config, not source ([`../01-architecture/directory-structure.md`](../01-architecture/directory-structure.md)).

## Secure data at rest
- Sensitive data (tokens, PII) → encrypted storage: **EncryptedSharedPreferences**
  / Android Keystore, or `react-native-keychain` / `expo-secure-store` equivalents.
- Never store secrets in plain AsyncStorage.
- Clear visitor PII on completion and on inactivity timeout.

## Network/transport
- HTTPS only; enable certificate pinning for the API.
- JWT/OAuth for auth; short-lived tokens, refresh server-side.
- Guard against XSS/injection if any WebView is embedded; disable JS unless required.

## Checklist
- [ ] Device Owner set & verified
- [ ] Status bar, recents, home escape blocked
- [ ] User restrictions (factory reset, safe boot) applied
- [ ] Auto-recover on crash/reboot
- [ ] Hidden PIN exit only
- [ ] Encrypted storage + HTTPS + cert pinning
- [ ] PII cleared on timeout
