---
created: 2026-04-01
tags: [concept]
related: [ [[State management MOC]], [[Local state in React]], [[Server state management]], [[Frontend system design MOC]] ]
status: active
---

# Global state management

## Overview

**Global state** is client-side data that needs to be accessible by many components across different parts of your application — without being passed down as props through every intermediary.

The canonical examples:
- Currently logged-in user info
- UI theme (dark/light mode)
- Sidebar open/closed
- Notification count
- Selected language

> **Critical distinction**: Global state tools (Zustand, Redux, Context) are for **client-owned state**. If the data comes from an API, use [[Server state management]] (TanStack Query) instead. Mixing these up is the most common state management mistake.

---

## Option 1: Context API (Built-in React)

React's built-in Context API is ideal for **low-frequency update data** (theme, auth session, language). It is not optimized for high-frequency updates because any change re-renders all consumers.

### Implementation: Theme Context

```typescript
// lib/context/ThemeContext.tsx
'use client'

import { createContext, useContext, useState, ReactNode } from 'react'

type Theme = 'light' | 'dark'
type ThemeContextType = { theme: Theme; toggleTheme: () => void }

const ThemeContext = createContext<ThemeContextType | null>(null)

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light')
  const toggleTheme = () => setTheme((prev) => (prev === 'light' ? 'dark' : 'light'))

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}

export function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) throw new Error('useTheme must be used within ThemeProvider')
  return context
}
```

```typescript
// app/layout.tsx — Wrap app with provider
import { ThemeProvider } from '@/lib/context/ThemeContext'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  )
}
```

```typescript
// Any component anywhere in the tree
'use client'
import { useTheme } from '@/lib/context/ThemeContext'

export default function ThemeToggle() {
  const { theme, toggleTheme } = useTheme()
  return <button onClick={toggleTheme}>Current: {theme}</button>
}
```

### When NOT to use Context
- State that changes frequently (every keystroke, every second) → causes too many re-renders.
- Large, complex state with many slices → Zustand or Redux is better organized.

---

## Option 2: Zustand (Recommended for Most Cases)

**Zustand** is a lightweight, flexible state management library. It is the best default choice for global client state in modern React apps.

**Why Zustand over Redux**: ~10x less boilerplate, no providers needed, simpler mental model, same performance.

### Installation

```bash
npm install zustand
```

### Basic Store: UI State

```typescript
// lib/stores/uiStore.ts
import { create } from 'zustand'

type UIStore = {
  isSidebarOpen: boolean
  activeModal: string | null
  toggleSidebar: () => void
  openModal: (id: string) => void
  closeModal: () => void
}

export const useUIStore = create<UIStore>((set) => ({
  isSidebarOpen: true,
  activeModal: null,
  toggleSidebar: () => set((state) => ({ isSidebarOpen: !state.isSidebarOpen })),
  openModal: (id) => set({ activeModal: id }),
  closeModal: () => set({ activeModal: null }),
}))
```

```typescript
// Usage in any component — no provider needed
'use client'
import { useUIStore } from '@/lib/stores/uiStore'

export default function Sidebar() {
  const { isSidebarOpen, toggleSidebar } = useUIStore()

  return (
    <aside className={isSidebarOpen ? 'open' : 'closed'}>
      <button onClick={toggleSidebar}>Toggle</button>
    </aside>
  )
}
```

### Advanced Zustand: With `persist` (localStorage)

```typescript
// lib/stores/settingsStore.ts
import { create } from 'zustand'
import { persist, createJSONStorage } from 'zustand/middleware'

type SettingsStore = {
  theme: 'light' | 'dark'
  language: string
  notificationsEnabled: boolean
  setTheme: (theme: 'light' | 'dark') => void
  setLanguage: (lang: string) => void
  toggleNotifications: () => void
}

export const useSettingsStore = create<SettingsStore>()(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      notificationsEnabled: true,
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleNotifications: () =>
        set((state) => ({ notificationsEnabled: !state.notificationsEnabled })),
    }),
    {
      name: 'user-settings', // localStorage key
      storage: createJSONStorage(() => localStorage),
    }
  )
)
```

### Zustand with Selectors (Prevent Unnecessary Re-renders)

```typescript
// ❌ This re-renders whenever ANY part of the store changes
const { isSidebarOpen, activeModal } = useUIStore()

// ✅ This only re-renders when `isSidebarOpen` changes
const isSidebarOpen = useUIStore((state) => state.isSidebarOpen)
const activeModal = useUIStore((state) => state.activeModal)
```

---

## Option 3: Redux Toolkit (For Complex, Event-Driven State)

**Redux Toolkit (RTK)** is the official, modern way to use Redux. It is the right choice for very complex applications with many different state slices, time-travel debugging needs, or when you need a standardized team approach.

Redux has more structure than Zustand — which is both its strength (predictable, auditable) and weakness (more boilerplate).

### When to use Redux Toolkit over Zustand
- Team of 10+ developers who need a strict, auditable pattern.
- Complex state with many interdependent slices.
- Need Redux DevTools for time-travel debugging.
- You already use RTK Query for API calls alongside it.

### Implementation

```bash
npm install @reduxjs/toolkit react-redux
```

```typescript
// lib/store/cartSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

type CartItem = { id: string; name: string; price: number; quantity: number }
type CartState = { items: CartItem[] }

const initialState: CartState = { items: [] }

export const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem: (state, action: PayloadAction<Omit<CartItem, 'quantity'>>) => {
      const existing = state.items.find((i) => i.id === action.payload.id)
      if (existing) {
        existing.quantity += 1 // Immer allows direct mutation — RTK uses Immer internally
      } else {
        state.items.push({ ...action.payload, quantity: 1 })
      }
    },
    removeItem: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter((i) => i.id !== action.payload)
    },
    clearCart: (state) => {
      state.items = []
    },
  },
})

export const { addItem, removeItem, clearCart } = cartSlice.actions
export default cartSlice.reducer
```

```typescript
// lib/store/store.ts
import { configureStore } from '@reduxjs/toolkit'
import cartReducer from './cartSlice'

export const store = configureStore({
  reducer: { cart: cartReducer },
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

---

## Comparison: Context vs Zustand vs Redux

| Dimension | Context API | Zustand | Redux Toolkit |
| :--- | :--- | :--- | :--- |
| **Bundle size** | 0 (built-in) | ~1KB | ~15KB |
| **Boilerplate** | Medium | Low | Medium-High |
| **Performance** | Poor for frequent updates | Excellent | Excellent |
| **DevTools** | None | Basic | Excellent (time-travel) |
| **Learning curve** | Low | Low | Medium |
| **Best for** | Theme, auth session | Most global state | Large teams, complex apps |

---

## Related Notes
- [[Local state in React]]
- [[Server state management]]
- [[Derived state and memoization]]
- [[State management MOC]]
