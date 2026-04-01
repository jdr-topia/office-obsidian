---
created: 2026-04-01
tags: [concept]
related: [ [[State management MOC]], [[Global state management]], [[Derived state and memoization]], [[Frontend system design MOC]] ]
status: active
---

# Local state in React

## Overview

**Local state** is state that belongs to a single component and is not shared with the rest of the application. It is the simplest and most common form of state in React.

The golden rule: **keep state as local as possible.** Only lift it up when multiple components genuinely need to share it.

React provides two built-in hooks for local state: `useState` for simple values, and `useReducer` for complex state logic.

---

## `useState` — The Foundation

`useState` is the primary hook for managing simple local state.

```typescript
import { useState } from 'react'

// Basic counter
export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount((prev) => prev - 1)}>Decrement</button>
    </div>
  )
}
```

### Functional Updates (Critical Pattern)

Always use the functional form `setCount(prev => prev + 1)` when the new state depends on the old state. This avoids stale closure bugs:

```typescript
// ❌ Stale closure bug — can lead to incorrect values with async updates
const handleIncrement = () => {
  setCount(count + 1)
  setCount(count + 1) // Still reads the same stale `count`!
  // Result: count incremented by 1, not 2
}

// ✅ Correct — uses the latest state value each time
const handleIncrement = () => {
  setCount((prev) => prev + 1)
  setCount((prev) => prev + 1) // Reads the updated prev from the previous call
  // Result: count incremented by 2
}
```

### Lazy Initialization

For expensive initial values, use a function to compute the initial state only once:

```typescript
// ❌ This runs on every render, even though useState only uses it on mount
const [data, setData] = useState(JSON.parse(localStorage.getItem('data')))

// ✅ Lazy init — the function is called ONLY on mount
const [data, setData] = useState(() => {
  return JSON.parse(localStorage.getItem('data') ?? 'null')
})
```

---

## Managing Object State

When state is an object, you must spread the existing state to avoid overwriting other fields:

```typescript
type UserProfile = {
  name: string
  email: string
  role: 'admin' | 'user'
}

export default function ProfileForm() {
  const [profile, setProfile] = useState<UserProfile>({
    name: 'Raman',
    email: 'raman@example.com',
    role: 'user',
  })

  const updateName = (name: string) => {
    setProfile((prev) => ({ ...prev, name })) // Spread to preserve other fields
  }

  const updateRole = (role: UserProfile['role']) => {
    setProfile((prev) => ({ ...prev, role }))
  }

  return (
    <form>
      <input value={profile.name} onChange={(e) => updateName(e.target.value)} />
      <select value={profile.role} onChange={(e) => updateRole(e.target.value as UserProfile['role'])}>
        <option value="user">User</option>
        <option value="admin">Admin</option>
      </select>
    </form>
  )
}
```

---

## `useReducer` — For Complex State Logic

`useReducer` is the right tool when your state has multiple sub-values that change in coordinated ways, or when the next state depends on complex logic involving the previous state.

Think of it as a mini Redux — you dispatch **actions** and a **reducer function** computes the next state.

### When to Prefer `useReducer` Over `useState`

| Scenario | Best Choice |
| :--- | :--- |
| One value (boolean, string, number) | `useState` |
| Object with 2-3 fields, simple updates | `useState` |
| Complex object with many fields | `useReducer` |
| Next state depends on complex logic | `useReducer` |
| Multiple state transitions share logic | `useReducer` |
| State has "actions" with business meaning | `useReducer` |

### Complete `useReducer` Example: Shopping Cart

```typescript
// types
type CartItem = { id: string; name: string; price: number; quantity: number }
type CartState = { items: CartItem[]; isOpen: boolean; couponCode: string | null }

type CartAction =
  | { type: 'ADD_ITEM'; payload: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; payload: { id: string } }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'APPLY_COUPON'; payload: { code: string } }
  | { type: 'TOGGLE_CART' }
  | { type: 'CLEAR_CART' }

// reducer — pure function, no side effects
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.items.find((i) => i.id === action.payload.id)
      if (existing) {
        return {
          ...state,
          items: state.items.map((i) =>
            i.id === action.payload.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
        }
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
      }
    }

    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter((i) => i.id !== action.payload.id),
      }

    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map((i) =>
          i.id === action.payload.id
            ? { ...i, quantity: Math.max(0, action.payload.quantity) }
            : i
        ),
      }

    case 'APPLY_COUPON':
      return { ...state, couponCode: action.payload.code }

    case 'TOGGLE_CART':
      return { ...state, isOpen: !state.isOpen }

    case 'CLEAR_CART':
      return { ...state, items: [], couponCode: null }

    default:
      return state
  }
}

// Component
const initialState: CartState = { items: [], isOpen: false, couponCode: null }

export default function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, initialState)

  const totalPrice = cart.items.reduce((sum, i) => sum + i.price * i.quantity, 0)

  return (
    <div>
      <button onClick={() => dispatch({ type: 'TOGGLE_CART' })}>
        Cart ({cart.items.length})
      </button>

      {cart.isOpen && (
        <div>
          {cart.items.map((item) => (
            <div key={item.id}>
              <span>{item.name}</span>
              <input
                type="number"
                value={item.quantity}
                onChange={(e) =>
                  dispatch({
                    type: 'UPDATE_QUANTITY',
                    payload: { id: item.id, quantity: Number(e.target.value) },
                  })
                }
              />
              <button onClick={() => dispatch({ type: 'REMOVE_ITEM', payload: { id: item.id } })}>
                Remove
              </button>
            </div>
          ))}
          <p>Total: ${totalPrice}</p>
          <button onClick={() => dispatch({ type: 'CLEAR_CART' })}>Clear</button>
        </div>
      )}
    </div>
  )
}
```

---

## Common Local State Patterns

### Toggle pattern

```typescript
const [isOpen, setIsOpen] = useState(false)
const toggle = () => setIsOpen((prev) => !prev)
```

### Form state pattern

```typescript
// For simple forms, local state is fine
// For complex forms, use Formik or React Hook Form
const [formData, setFormData] = useState({ name: '', email: '' })

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setFormData((prev) => ({ ...prev, [e.target.name]: e.target.value }))
}
```

### Previous state pattern

```typescript
// Track previous value for "undo" functionality
const [value, setValue] = useState('')
const [history, setHistory] = useState<string[]>([])

const handleChange = (newValue: string) => {
  setHistory((prev) => [...prev, value]) // Save current before overwriting
  setValue(newValue)
}

const handleUndo = () => {
  setValue(history[history.length - 1] ?? '')
  setHistory((prev) => prev.slice(0, -1))
}
```

---

## When Local State Is Not Enough

Lift state up to the closest common ancestor when:
- Two sibling components need to read the same state.
- A parent needs to control a child's state.

When even "lifting up" becomes unwieldy, reach for:
→ [[Global state management]] (Zustand, Context)
→ [[Server state management]] (TanStack Query) for API data
→ [[Derived state and memoization]] for computed values

---

## Related Notes
- [[State management MOC]]
- [[Global state management]]
- [[Derived state and memoization]]
- [[Server state management]]
