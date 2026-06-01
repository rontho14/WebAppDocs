# Strict TypeScript

Strict from day one — catch errors before they reach a deployed kiosk.

## tsconfig
```jsonc
{
  "compilerOptions": {
    "strict": true,                 // all strict flags on
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "baseUrl": "./",
    "paths": { "@/*": ["./src/*"] } // path alias -> no ../../../ imports
  }
}
```
> Also configure the alias in `babel.config.js` via
> `babel-plugin-module-resolver` so Metro resolves `@/` at runtime.

## Shared types
```ts
// src/types/common.ts
export interface User {
  id: number;
  username: string;
  role: 'admin' | 'user';
}

export interface Visitor {
  id: string;
  name: string;
  status: 'pending' | 'checked-in' | 'checked-out';
  assignedTo?: User;
}
```

## Conventions
- No `any`. Use `unknown` + narrowing for uncertain data.
- Type API responses; never trust raw JSON — validate at the boundary (Zod) and
  infer types: `type Visitor = z.infer<typeof visitorSchema>`.
- Prefer union literals (`'pending' | 'done'`) over loose `string`.
- Type navigation params and Redux state/dispatch (see linked architecture docs).
- Derive types from a single source of truth; don't duplicate shapes.
- `as const` for token objects (colors, spacing) to keep literal types.

## Why strict for kiosks
A runtime type error on an unattended device = a white screen no one can fix.
Strict typing + schema validation prevents the most common production crashes.
