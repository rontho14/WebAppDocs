# State Management

Match the tool to the kind of state. **Do not put server data in Redux.**

| State kind | Example | Tool |
|-----------|---------|------|
| Local UI | input value, modal open | `useState` / `useReducer` |
| Shared client | auth, kiosk status, theme | Redux Toolkit **or** Context |
| Server cache | API responses, lists | React Query |

## Local
```tsx
const [open, setOpen] = useState(false);
// complex transitions:
const [state, dispatch] = useReducer(reducer, initial);
```

## Global client state — Redux Toolkit
```ts
// store/kioskSlice.ts
import { createSlice } from '@reduxjs/toolkit';

const kioskSlice = createSlice({
  name: 'kiosk',
  initialState: { locked: false },
  reducers: {
    lock: (s) => { s.locked = true; },
    unlock: (s) => { s.locked = false; },
  },
});
export const { lock, unlock } = kioskSlice.actions;
export default kioskSlice.reducer;

// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
export const store = configureStore({ reducer: { kiosk: kioskReducer } });
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```
Typed hooks (always use these, never raw `useSelector`):
```ts
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

## Global client state — Context (lightweight)
Good for theme/auth/locale where updates are infrequent.
```tsx
const ThemeContext = createContext<ThemeCtx | null>(null);
export const ThemeProvider = ({ children }: PropsWithChildren) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  return <ThemeContext.Provider value={{ theme, setTheme }}>{children}</ThemeContext.Provider>;
};
```
⚠️ Context re-renders all consumers on change — don't use it for high-frequency state.

## Server state — React Query
Handles caching, retries, background refresh, pagination. See
[`service-layer.md`](service-layer.md).

## Rules
- Lift state only as high as needed.
- Persist kiosk/auth flags with `redux-persist` + encrypted storage.
- Alternatives (Zustand, MobX, Recoil) are acceptable but standardize on one.
