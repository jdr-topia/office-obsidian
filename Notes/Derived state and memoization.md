---
created: 2026-04-01
tags: [concept]
related: [ [[State management MOC]], [[Local state in React]], [[Global state management]], [[Performance requirements]], [[Frontend system design MOC]] ]
status: active
---

# Derived state and memoization

## Overview

**Derived state** is state that is **computed from other state** rather than stored independently. **Memoization** is the technique of caching an expensive computation so it doesn't re-run if its inputs haven't changed.

Together, these concepts are critical for:
1. Avoiding redundant state that can get out of sync.
2. Preventing unnecessary re-renders in React.
3. Keeping component logic clean and predictable.

---

## Part 1: Derived State

### The Anti-Pattern: Redundant Stored State

```typescript
// ❌ Anti-pattern — storing derived data as state
function CartSummary() {
  const [items, setItems] = useState<CartItem[]>([])
  const [totalPrice, setTotalPrice] = useState(0)      // Redundant!
  const [itemCount, setItemCount] = useState(0)         // Redundant!

  const addItem = (item: CartItem) => {
    const newItems = [...items, item]
    setItems(newItems)
    setTotalPrice(newItems.reduce((sum, i) => sum + i.price, 0)) // Must sync manually
    setItemCount(newItems.length)                                 // Must sync manually
  }
  // If any sync step is missed, the UI shows inconsistent data
}
```

### The Correct Pattern: Compute From Source of Truth

```typescript
// ✅ Correct — derive totals from the single source of truth
function CartSummary() {
  const [items, setItems] = useState<CartItem[]>([])

  // These are computed on every render, always consistent
  const totalPrice = items.reduce((sum, i) => sum + i.price * i.quantity, 0)
  const itemCount = items.reduce((sum, i) => sum + i.quantity, 0)
  const hasExpressEligibleItems = items.some((i) => i.isExpressEligible)

  // No need to sync multiple state values — totals always reflect `items`
}
```

**Rule**: If a value can be computed from existing state or props, **do not store it in state**. Compute it directly.

---

## Part 2: Memoization in React

Computation is cheap in most cases. But for **expensive calculations** or for **preventing unnecessary child re-renders**, React provides memoization tools.

### When memoization is needed (and when it's not)

```typescript
// ❌ Do NOT memoize this — the calculation is trivial
const total = useMemo(() => items.length * 2, [items]) // Overkill

// ✅ DO memoize this — genuinely expensive: sorting 10,000 items
const sortedItems = useMemo(() => {
  return [...items].sort((a, b) => a.price - b.price)
}, [items])
```

> **Memoization is not free.** It adds memory overhead and cognitive complexity. Only use it when you have a measured performance problem.

---

### `useMemo` — Memoizing Computed Values

`useMemo` caches the result of a function. It only re-runs when its dependencies change.

```typescript
'use client'
import { useMemo, useState } from 'react'

type Product = { id: string; name: string; price: number; category: string }

function ProductList({ products }: { products: Product[] }) {
  const [searchQuery, setSearchQuery] = useState('')
  const [selectedCategory, setSelectedCategory] = useState('all')

  // Memoized: only re-filters when products, searchQuery, or selectedCategory changes
  const filteredProducts = useMemo(() => {
    console.log('Filtering products...') // See when this re-runs Filters

    return products
      .filter((p) => selectedCategory === 'all' || p.category === selectedCategory)
      .filter((p) => p.name.toLowerCase().includes(searchQuery.toLowerCase()))
  }, [products, searchQuery, selectedCategory])

  // Memoized: only re-computes when filteredProducts changes
  const totalValue = useMemo(
    () => filteredProducts.reduce((sum, p) => sum + p.price, 0),
    [filteredProducts]
  )

  return (
    <div>
      <input
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Search..."
      />
      <p>Total value: ${totalValue}</p>
      <p>Showing {filteredProducts.length} products</p>
      {filteredProducts.map((p) => <ProductCard key={p.id} product={p} />)}
    </div>
  )
}
```

---

### `useCallback` — Memoizing Functions

`useCallback` caches a function reference. Without it, every render creates a **new function reference**, which causes child components (even `React.memo` ones) to re-render unnecessarily.

```typescript
'use client'
import { useCallback, useState } from 'react'

function ParentList() {
  const [items, setItems] = useState<string[]>(['Item A', 'Item B'])

  // ❌ Without useCallback: new function reference on every ParentList render
  // const handleDelete = (item: string) => setItems((prev) => prev.filter((i) => i !== item))

  // ✅ With useCallback: same function reference across renders
  // Only re-creates when setItems changes (which it never does)
  const handleDelete = useCallback((item: string) => {
    setItems((prev) => prev.filter((i) => i !== item))
  }, []) // Empty deps because setItems is stable

  return (
    <ul>
      {items.map((item) => (
        <ListItem key={item} item={item} onDelete={handleDelete} /> // Memoized child won't re-render
      ))}
    </ul>
  )
}
```

---

### `React.memo` — Preventing Child Component Re-renders

`React.memo` wraps a component to prevent it from re-rendering if its props haven't changed (shallow comparison):

```typescript
import { memo, useCallback } from 'react'

// This component will ONLY re-render when `item` or `onDelete` props change
const ListItem = memo(function ListItem({
  item,
  onDelete,
}: {
  item: string
  onDelete: (item: string) => void
}) {
  console.log(`Rendering: ${item}`)
  return (
    <li>
      {item}
      <button onClick={() => onDelete(item)}>Delete</button>
    </li>
  )
})
```

> `React.memo` only works correctly when the parent uses `useCallback` for function props. Without `useCallback`, the function reference changes every render, making `React.memo` useless for that prop.

---

### Selectors in Zustand (Derived Global State)

When using Zustand, selectors allow you to derive computed values from the store without storing them:

```typescript
// lib/stores/cartStore.ts
import { create } from 'zustand'

type CartItem = { id: string; price: number; quantity: number }
type CartStore = {
  items: CartItem[]
  addItem: (item: CartItem) => void
}

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}))

// Selector — derives total from store state
// Only re-renders when the computed value changes, not on any store change
export const useCartTotal = () =>
  useCartStore((state) => state.items.reduce((sum, i) => sum + i.price * i.quantity, 0))

export const useCartItemCount = () =>
  useCartStore((state) => state.items.reduce((sum, i) => sum + i.quantity, 0))
```

```typescript
// Usage — only re-renders this component when the TOTAL changes
function CartBadge() {
  const total = useCartTotal()
  const count = useCartItemCount()
  return <span>{count} items — ${total}</span>
}
```

---

## The Memoization Checklist

Before adding `useMemo` or `useCallback`, ask:

- [ ] Is this computation actually slow? (Profile in DevTools first)
- [ ] Is this component re-rendering too often? (Use React DevTools Profiler)
- [ ] Is the child component wrapped in `React.memo`? (Otherwise `useCallback` helps nothing)
- [ ] Does the function/value have stable dependencies?

> **Most of the time, you don't need memoization.** React is fast. Premature memoization adds complexity without measurable benefit. Profile first, optimize second.

---

## Related Notes
- [[Local state in React]]
- [[Global state management]]
- [[State management MOC]]
- [[Performance requirements]]
