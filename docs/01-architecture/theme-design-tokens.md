# Theme & Design Tokens

Single source of truth for colors, type, and spacing. Import tokens everywhere;
never hardcode hex/sizes in components. Keeps the UI consistent across screens
and devices.

## Tokens
```ts
// src/theme/index.ts
import { responsive } from '@/utils/responsive'; // fontSize scales by width

export const colors = {
  primary: { 50: '#eff6ff', 500: '#3b82f6', 700: '#1d4ed8' },
  gray:    { 100: '#f3f4f6', 500: '#6b7280', 900: '#111827' },
} as const;

export const typography = {
  sizes: {
    xs: responsive.fontSize(12),
    sm: responsive.fontSize(14),
    base: responsive.fontSize(16),
    lg: responsive.fontSize(18),
    xl: responsive.fontSize(20),
  },
  weights: { normal: '400', medium: '500', bold: '700' },
} as const;

export const spacing = { xs: 4, sm: 8, md: 16, lg: 24, xl: 32 } as const;
```
`as const` keeps literal types (see [`../05-quality/typescript-strict.md`](../05-quality/typescript-strict.md)).

## Touch tokens
Keep tap-size constants here too, referenced by [`../02-tablet-ux/touch-targets.md`](../02-tablet-ux/touch-targets.md):
```ts
// src/theme/touch.ts
export const TOUCH = { min: 48, comfortable: 56, primary: 64, gap: 8 } as const;
```

## Consuming tokens
```tsx
const styles = StyleSheet.create({
  card: { padding: spacing.md, backgroundColor: colors.gray[100], borderRadius: 12 },
  title: { fontSize: typography.sizes.lg, fontWeight: typography.weights.bold, color: colors.gray[900] },
});
```

## Light/dark via Context
Expose the active theme through Context so screens react to mode changes
(see [`state-management.md`](state-management.md) for the Context pattern):
```tsx
const ThemeContext = createContext<{ mode: 'light' | 'dark'; colors: typeof colors }>(/* ... */);
export const useTheme = () => useContext(ThemeContext);
```

## Rules
- One token file; extend it, don't redefine values inline.
- Scale font sizes through `responsive.fontSize` so type adapts to screen width.
- Spacing is a fixed scale (4/8/16/24/32) — pick a token, don't invent values.
- High contrast for kiosks used under bright/ambient light.
