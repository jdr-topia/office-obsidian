---
created: 2026-04-10
tags: [typescript, dx, type-safety]
related: [ [[Developer experience (DX) MOC]], [[Unit testing with Jest and Vitest]] ]
status: active
---

# TypeScript in frontend

TypeScript is a statically typed superset of JavaScript that compiles to plain JavaScript. It is the single most impactful tool for improving code quality and developer confidence in a large frontend codebase.

## Why TypeScript?

| Problem in JS | TypeScript Solution |
|---|---|
| `undefined is not a function` at runtime | Type error at compile time, before shipping |
| Refactoring breaks callsites silently | Compiler flags every broken callsite instantly |
| No IDE autocomplete for API shapes | Full IntelliSense on all objects and functions |
| Inconsistent function signatures | `interface` and `type` enforce contracts |
| "What does this function return?" | Return types are self-documenting |

---

## Core Concepts for Frontend

### 1. Typing Component Props

```tsx
// Define a clear contract for every component
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'ghost';
  onClick: () => void;
  disabled?: boolean;
}

export const Button = ({ label, variant = 'primary', onClick, disabled }: ButtonProps) => (
  <button className={`btn btn-${variant}`} onClick={onClick} disabled={disabled}>
    {label}
  </button>
);
```

### 2. Typing API Responses

```ts
// Define the shape of data coming from your API
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'editor' | 'viewer';
  createdAt: string; // ISO date string
}

async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`Failed to fetch user ${id}`);
  return res.json() as Promise<User>;
}
```

### 3. Discriminated Unions (Replacing String Enums)

Model state as a discriminated union to get exhaustive type checking:

```ts
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function renderUser(state: RequestState<User>) {
  switch (state.status) {
    case 'loading': return <Spinner />;
    case 'error':   return <ErrorMessage message={state.error} />;
    case 'success': return <UserCard user={state.data} />; // data is typed as User here
    case 'idle':    return null;
  }
}
```

### 4. Utility Types

```ts
// Pick only specific fields from a type
type UserPreview = Pick<User, 'id' | 'name'>;

// Make all fields optional (for partial updates/patches)
type UserPatch = Partial<User>;

// Make all fields required
type RequiredUser = Required<User>;

// Omit specific fields
type UserWithoutId = Omit<User, 'id'>;

// Extract the type from a Promise
type ResolvedUser = Awaited<ReturnType<typeof fetchUser>>;
```

---

## `tsconfig.json` — Key Settings

```json
{
  "compilerOptions": {
    "strict": true,            // Enables all strict checks — ALWAYS turn this on
    "noUncheckedIndexedAccess": true, // arr[0] is T | undefined, not T
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "paths": {
      "@/*": ["./src/*"]       // Path aliases to avoid ../../../ hell
    }
  }
}
```

---

## Anti-Patterns to Avoid

```ts
// ❌ Never use `any` — it turns off TypeScript completely
const data: any = await fetch('/api/data');

// ✅ Use `unknown` and narrow the type
const data: unknown = await fetch('/api/data');
if (isUser(data)) { /* now data is typed */ }

// ❌ Casting with `as` when you're not certain
const user = response as User;

// ✅ Use a type guard or validator (e.g., zod)
import { z } from 'zod';
const UserSchema = z.object({ id: z.string(), name: z.string() });
const user = UserSchema.parse(response); // Throws at runtime if shape is wrong
```

---

## Related Topics
- [[Developer experience (DX) MOC]]
- [[Linting and code formatting]]
- [[Unit testing with Jest and Vitest]]
