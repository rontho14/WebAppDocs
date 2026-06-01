# Architecture Patterns

Pick the lightest pattern that solves the problem. Default to **feature-based +
hooks for logic + presentational components for UI**.

## Component-based (foundation)
Break the UI into small, reusable, single-responsibility components. Compose
screens from them. Minimize duplication.

## Container / Presentational
Separate *logic* from *rendering*. In modern RN, the "container" is usually a
**custom hook**, not a wrapper component.

```tsx
// logic (container)  -> useCheckin.ts
export function useCheckin() {
  const [visitor, setVisitor] = useState<Visitor | null>(null);
  const submit = useCallback(async (data: VisitorInput) => {
    setVisitor(await checkinService.create(data));
  }, []);
  return { visitor, submit };
}

// presentational -> CheckinScreen.tsx
export function CheckinScreen() {
  const { visitor, submit } = useCheckin();   // logic injected
  return <CheckinForm onSubmit={submit} done={!!visitor} />;
}
```
Presentational components take props, render UI, hold no business logic.

## Atomic Design (for the design system)
Layer the shared UI: **atoms** (Text, Icon) → **molecules** (LabeledInput) →
**organisms** (CheckinForm) → **templates** → **screens**. Keep atoms/molecules
in `src/components/`, organisms near their feature.

## Flux / Redux (predictable global state)
For app-wide state use a unidirectional flow: `action → reducer → store → view`.
Prefer **Redux Toolkit**. See [`state-management.md`](state-management.md).

## Decision guide
| Need | Use |
|------|-----|
| Reusable UI piece | Component-based + atomic |
| Screen logic | Custom hook (container) |
| Server data | React Query (not Redux) |
| App-wide client state (auth, kiosk, theme) | Redux Toolkit or Context |
| One small global value | Context API |
