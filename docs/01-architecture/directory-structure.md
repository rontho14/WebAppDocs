# Directory Structure

Feature-oriented layout. Group by domain first, by type second.

```
src/
├── components/      # Shared, reusable UI (Button, Card, Modal)
├── features/        # Self-contained domains (each owns its slice of everything)
│   └── checkin/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       ├── store/        # feature slice
│       └── screens/
├── screens/         # App-level / cross-feature full-screen views
├── navigation/      # Navigators + route types (React Navigation)
├── store/           # Root store config + global slices
├── services/        # Cross-cutting API clients (apiClient, authService)
├── hooks/           # Global custom hooks (useDeviceInfo, useKiosk)
├── utils/           # Pure helpers (responsive, formatters)
├── theme/           # colors, typography, spacing tokens
├── types/           # Shared TS types/interfaces
├── assets/          # Images, fonts, Lottie
└── config/          # env access, constants
```

## Native modules (provisional architecture)
Architecture is **not finalized**: the working assumption is **React Native + TS
for all UI/UX**, with **native code (Kotlin or Java) for device support** that JS
can't reach — kiosk Device Owner / Lock Task, some peripherals, low-latency input.
Native code lives under the Android project, not `src/`:
```
android/app/src/main/java/com/yourapp/   # native modules, receivers (Kotlin/Java)
res/xml/                                  # device_admin.xml, etc.
src/services/kiosk.ts                     # thin JS wrapper over the native module
```
Keep the boundary clean: native module exposes a minimal API; all logic/UI stays
in RN/TS. See [`../04-kiosk/lock-task-mode.md`](../04-kiosk/lock-task-mode.md).

## Rules

- A folder graduates to `features/<name>/` once it owns 2+ screens or its own state.
- Truly shared code lives at `src/` root (`components/`, `hooks/`, `utils/`).
- No deep relative imports (`../../../`). Use the `@/` alias.
- One default export per file; name the file after the component/hook.
- Co-locate tests: `Button.tsx` ➝ `Button.test.tsx`.

## Environment config

Never hardcode endpoints/secrets. Use `react-native-config` (`.env` files):

```ts
// src/config/env.ts
import Config from 'react-native-config';

export const env = {
  apiUrl: Config.API_URL!,
  kioskPin: Config.KIOSK_EXIT_PIN!,
};
```

`.env`, `.env.staging`, `.env.production` — never commit real secrets.
