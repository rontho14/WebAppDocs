# Navigation & Menus

Use **React Navigation**. Choose the navigator by screen size and kiosk flow.

## Navigator choice
| Navigator | Use for |
|-----------|---------|
| **Stack** | Linear flows (check-in steps), modals |
| **Bottom Tabs** | 3–5 top-level sections on phones |
| **Drawer** | Secondary/admin actions, hidden in kiosk |
| **Navigation Rail** (custom / left tabs) | Tablet primary nav — persistent side bar |

> On tablets prefer a **persistent left rail** over bottom tabs: it suits
> landscape and keeps content centered. Android's large-screen quality
> guidelines explicitly call for **navigation rails to replace bottom bars on
> large screens** for ergonomics, and for **context menus to appear next to the
> selected item** (not full-screen). Source: [`../references.md`](../references.md).

## Typed routes
```ts
// navigation/types.ts
export type RootStackParamList = {
  Home: undefined;
  VisitorDetail: { visitorId: string };
  AdminPin: undefined;
};
declare global {
  namespace ReactNavigation { interface RootParamList extends RootStackParamList {} }
}
```

## Responsive navigator
```tsx
export function RootNavigator() {
  const { width } = useWindowDimensions();
  return width >= 840 ? <RailNavigator /> : <TabNavigator />;
}
```

## Navigation rail sketch (tablet primary nav)
A persistent vertical rail beside the content (replaces the bottom bar on large screens):
```tsx
const ITEMS = [
  { key: 'Home', icon: 'home' },
  { key: 'Visitors', icon: 'people' },
] as const;

function RailNavigator() {
  const [active, setActive] = useState<string>('Home');
  return (
    <View style={{ flex: 1, flexDirection: 'row' }}>
      <View style={{ width: 88, paddingVertical: spacing.md, alignItems: 'center' }}>
        {ITEMS.map((it) => (
          <Pressable
            key={it.key}
            onPress={() => setActive(it.key)}
            accessibilityRole="tab"
            accessibilityState={{ selected: active === it.key }}
            style={{ minHeight: TOUCH.comfortable, alignItems: 'center', marginBottom: spacing.lg }}>
            <Icon name={it.icon} color={active === it.key ? colors.primary[500] : colors.gray[500]} />
            <Text>{it.key}</Text>
          </Pressable>
        ))}
      </View>
      <View style={{ flex: 1 }}>{/* active screen */}</View>
    </View>
  );
}
```
Tokens (`spacing`, `colors`, `TOUCH`) come from [`../01-architecture/theme-design-tokens.md`](../01-architecture/theme-design-tokens.md).

## Kiosk navigation rules
- **No drawer/back exit to system.** Hide hardware-style back where it leaves the flow.
- Admin/settings screens reachable only via a **hidden gesture + PIN**
  (see [`../04-kiosk/security-config.md`](../04-kiosk/security-config.md)).
- Keep depth shallow (≤ 3 levels) — visitors get lost otherwise.
- Always show where the user is and how to return to Home.
- Auto-reset to Home after an inactivity timeout (clear PII, return to idle screen).
- Disable deep links / external intents that could break the lock.
