---
name: Zustand Store Implementation
description: Sets up a global client-side state store using Zustand.
---

# Zustand Store Implementation Skill

This skill standardize how global client state is managed in the application.

## Usage

When the user asks to "add global state", "create a store", or "manage state across components", follow these steps.

## Steps

1.  **Create Store File**:
    - Create `context/store/[feature]Store.ts`.
2.  **Define Interface**:
    - Create an interface `[Feature]State` defining data properties.
    - Create an interface `[Feature]Actions` defining modifier methods.
    - Combine them: `type [Feature]Store = [Feature]State & [Feature]Actions`.
3.  **Implement Store**:
    - Import `create` from `zustand`.
    - Use `create<[Feature]Store>((set, get) => ({ ... }))`.
    - Implement actions using `set` (for merging state) and `get` (for reading current state).
4.  **Usage**:
    - In components, import the hook: `const { data, update } = use[Feature]Store()`.

## Example Code

```typescript
import { create } from "zustand";

interface CounterState {
  count: number;
}

interface CounterActions {
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

type CounterStore = CounterState & CounterActions;

export const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```
