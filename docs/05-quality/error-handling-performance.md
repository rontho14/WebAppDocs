# Error Handling & Performance

A kiosk runs unattended for days — it must never show a white screen and must
stay smooth.

## Error boundaries (prevent white screens)
```tsx
class ErrorBoundary extends React.Component<PropsWithChildren, { hasError: boolean }> {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(err: Error, info: React.ErrorInfo) {
    logError(err, info);            // ship to Sentry/LogRocket
  }
  render() {
    return this.state.hasError
      ? <FallbackScreen onReset={() => { this.setState({ hasError: false }); restartKioskFlow(); }} />
      : this.props.children;
  }
}
```
- Wrap the app root **and** risky subtrees.
- Fallback should **return to the idle/home kiosk screen**, never exit the app.

## Centralized error normalization
```ts
export function normalizeError(err: unknown): AppError {
  if (axios.isAxiosError(err)) return { kind: 'network', status: err.response?.status, message: err.message };
  return { kind: 'unknown', message: String(err) };
}
```

## Logging & monitoring
- Integrate **Sentry** (crashes) / **LogRocket** for remote visibility — you
  can't watch the device.
- Log kiosk lifecycle events (enter/exit lock, recovery, peripheral errors).
- Add a heartbeat/health ping so a frozen kiosk is detectable remotely.

## Performance techniques
| Technique | Use |
|-----------|-----|
| `React.memo` | skip re-render when props equal |
| `useMemo` | memo expensive computation |
| `useCallback` | stable callback identity for memo'd children |
| `React.lazy` + `Suspense` | code-split heavy screens |
| `FlatList`/`FlashList` | virtualize long lists |
| Image caching | `react-native-fast-image`; size to display |
| Lottie | optimized animations vs heavy GIFs |

```tsx
const TaskCard = React.memo(({ task }: { task: Task }) => { /* ... */ });
const onPress = useCallback(() => submit(id), [id]);
```

## Long-running kiosk concerns
- Watch for memory leaks (uncleared timers/listeners — always clean up in `useEffect`).
- Auto-reload/reset state on inactivity to release accumulated memory.
- Avoid unbounded in-memory caches; cap React Query cache.

## Deployment / CI-CD
- Build with native CLI; sign & distribute via Play Store (managed/private) or EMM.
- CI (GitHub Actions): lint → typecheck → unit tests → build on every push.
- Roll out via EMM/MDM to push updates to the fleet; pin app version per device.
