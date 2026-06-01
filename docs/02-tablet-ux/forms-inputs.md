# Forms & Inputs

Kiosk forms are filled on a touchscreen, often via the on-screen keyboard. Keep
them short, forgiving, and large.

## Input field
```tsx
<TextInput
  style={styles.input}            // minHeight 56, fontSize >= 18
  keyboardType="email-address"     // right keyboard per field
  textContentType="emailAddress"
  autoCapitalize="none"
  autoCorrect={false}
  returnKeyType="next"
  blurOnSubmit={false}
  placeholderTextColor="#999"
  accessibilityLabel="Email"
/>
```

## Keyboard handling
```tsx
<KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : undefined} style={{ flex: 1 }}>
  <ScrollView keyboardShouldPersistTaps="handled">
    {/* fields */}
  </ScrollView>
</KeyboardAvoidingView>
```
- Pick `keyboardType` per field: `numeric`, `phone-pad`, `email-address`.
- `returnKeyType="next"` + focus refs to chain fields; `"done"` on the last.
- Hardware keyboards/scanners may be attached — handle `onSubmitEditing`.

## Validation
Use a schema (Zod / Yup) + `react-hook-form`. Validate on blur/submit, not per keystroke.
```ts
const schema = z.object({
  name: z.string().min(2, 'Required'),
  email: z.string().email('Invalid email'),
});
type FormData = z.infer<typeof schema>;
```
- Show errors inline, near the field, in plain language.
- Disable submit while invalid or pending; show a spinner on submit.

## Gestures — keep them basic
NN/g tablet research: **advanced gestures effectively don't exist** to most
users — they reliably use only **tap, press, swipe, drag, pinch**. Users also
"can't see the gesture they just made" or what's tappable. So:
- Never hide a required action behind a non-obvious gesture; provide a visible control.
- Make tappable elements obviously tappable (avoid flat, undifferentiated UI).
- Source: [`../references.md`](../references.md).

## Kiosk-specific rules
- Minimize fields — every field is friction for a standing visitor.
- Large tap targets (see [`touch-targets.md`](touch-targets.md)).
- Prefer pickers/toggles/segmented controls over free text where possible.
- Clear all input on completion/timeout (privacy — no leftover PII).
- Block autofill suggestions that leak the previous visitor's data
  (`autoComplete="off"`, `importantForAutofill="no"`).
- Confirm destructive/irreversible actions.
