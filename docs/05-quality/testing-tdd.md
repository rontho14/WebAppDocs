# Testing & TDD

Unattended kiosks can't be hot-fixed by a user — test before shipping.

## Test pyramid
| Level | Tool | Scope |
|-------|------|-------|
| Unit | **Jest** | pure logic, reducers, hooks, utils |
| Component | **React Native Testing Library** | render + interaction |
| API mocking | **MSW** | intercept network in tests |
| E2E | **Detox** | full flows on device/emulator |

## TDD loop (Red → Green → Refactor)
1. **Red** — write a failing test describing the desired behavior.
2. **Green** — write the minimum code to pass.
3. **Refactor** — clean up with the test as a safety net.
Repeat per small increment. Write the test *first* for new logic and bug fixes
(a bug fix starts with a failing test that reproduces it).

## Unit example
```ts
test('lock sets locked = true', () => {
  expect(kioskReducer({ locked: false }, lock())).toEqual({ locked: true });
});
```

## Component example (RNTL)
```tsx
test('submits visitor on press', async () => {
  const onSubmit = jest.fn();
  const { getByText, getByLabelText } = render(<CheckinForm onSubmit={onSubmit} />);
  fireEvent.changeText(getByLabelText('Email'), 'a@b.com');
  fireEvent.press(getByText('Check in'));
  await waitFor(() => expect(onSubmit).toHaveBeenCalled());
});
```

## API mocking (MSW)
```ts
const server = setupServer(
  http.get('*/visitors/:id', () => HttpResponse.json({ id: '1', name: 'Ada' })),
);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## E2E (Detox) — cover critical kiosk flows
- App launches and enters kiosk mode.
- Full check-in happy path.
- Hidden admin PIN exit works; wrong PIN is rejected.
- Recovery after relaunch.

## Rules
- Test behavior, not implementation details.
- Mock the native kiosk module in unit tests; verify the real thing in Detox/manual device runs.
- Run unit + lint + typecheck in CI on every push (see deploy notes in
  [`error-handling-performance.md`](error-handling-performance.md)).
- Keep tests co-located (`Component.test.tsx`).
