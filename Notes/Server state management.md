---
created: 2026-04-01
tags: [concept]
related: [ [[State management MOC]], [[Global state management]], [[Data fetching MOC]], [[REST API integration]], [[Frontend system design MOC]] ]
status: active
---

# Server state management

## Overview

**Server state** is data that **lives on the server** and is fetched, displayed, and mutated by your frontend. It is fundamentally different from client state because it is not owned by your app — it can change at any time due to other users, background jobs, or server events.

Server state has unique challenges:
- It can become **stale** (out of sync with the server).
- It requires **caching** to avoid redundant network requests.
- It needs **loading and error states**.
- It needs **refetching** when the user returns to a page.
- Mutations must be coordinated with cache invalidation.

**TanStack Query** (formerly React Query) and **SWR** are purpose-built tools for managing server state.

> The golden rule: **Never put API data in Redux/Zustand.** Use a server state library instead.

---

## TanStack Query (Recommended)

TanStack Query is the industry standard for server state management. It provides caching, background refetching, synchronization, and mutation management out of the box.

### Setup

```bash
npm install @tanstack/react-query
```

```typescript
// app/providers.tsx — Wrap your app with QueryClientProvider
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { useState } from 'react'

export default function Providers({ children }: { children: React.ReactNode }) {
  // Create a new QueryClient per session to avoid shared state between users (SSR safe)
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute — data is "fresh" for 1 minute
            retry: 2,             // Retry failed requests twice
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

```typescript
// app/layout.tsx
import Providers from './providers'
export default function RootLayout({ children }) {
  return <html><body><Providers>{children}</Providers></body></html>
}
```

---

### `useQuery` — Fetching Data

```typescript
// hooks/useOrders.ts
import { useQuery } from '@tanstack/react-query'

type Order = { id: string; status: string; total: number }

async function fetchOrders(): Promise<Order[]> {
  const res = await fetch('/api/orders')
  if (!res.ok) throw new Error('Failed to fetch orders')
  return res.json()
}

export function useOrders() {
  return useQuery({
    queryKey: ['orders'],         // Unique cache key — same key anywhere = same cache
    queryFn: fetchOrders,
    staleTime: 30_000,            // Data is fresh for 30 seconds — no refetch during this window
    gcTime: 5 * 60 * 1000,        // Keep unused data in cache for 5 minutes
  })
}
```

```typescript
// app/dashboard/page.tsx
'use client'
import { useOrders } from '@/hooks/useOrders'

export default function OrdersPage() {
  const { data: orders, isLoading, isError, error } = useOrders()

  if (isLoading) return <OrdersSkeleton />
  if (isError) return <p>Error: {error.message}</p>

  return (
    <table>
      {orders.map((order) => (
        <tr key={order.id}>
          <td>{order.id}</td>
          <td>{order.status}</td>
          <td>${order.total}</td>
        </tr>
      ))}
    </table>
  )
}
```

### Query Keys — The Cache Key System

Query keys are arrays that uniquely identify a query. TanStack Query uses them for caching, deduplication, and invalidation:

```typescript
// Simple key
useQuery({ queryKey: ['orders'] })  // GET /api/orders (all orders)

// Parameterized key — separate cache entry per ID
useQuery({ queryKey: ['orders', orderId] })  // GET /api/orders/:orderId

// Multiple parameters
useQuery({
  queryKey: ['orders', { status: 'pending', page: 2 }],
})

// When orderId changes, TanStack Query automatically refetches
function OrderDetail({ orderId }: { orderId: string }) {
  const { data } = useQuery({
    queryKey: ['orders', orderId],
    queryFn: () => fetch(`/api/orders/${orderId}`).then((r) => r.json()),
  })
  return <div>{data?.status}</div>
}
```

---

### `useMutation` — Creating, Updating, Deleting Data

```typescript
// hooks/useCreateOrder.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'

type CreateOrderInput = { productId: string; quantity: number }

async function createOrder(input: CreateOrderInput) {
  const res = await fetch('/api/orders', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(input),
  })
  if (!res.ok) throw new Error('Failed to create order')
  return res.json()
}

export function useCreateOrder() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: createOrder,

    // On success: invalidate the orders cache so it refetches
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['orders'] })
    },

    onError: (error) => {
      console.error('Order creation failed:', error)
    },
  })
}
```

```typescript
// Component usage
'use client'
import { useCreateOrder } from '@/hooks/useCreateOrder'

export default function OrderForm() {
  const { mutate: createOrder, isPending, isError } = useCreateOrder()

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    createOrder({ productId: 'abc123', quantity: 2 })
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Order'}
      </button>
      {isError && <p>Failed to create order. Try again.</p>}
    </form>
  )
}
```

---

### Optimistic Updates

Optimistic updates make the UI feel instant by updating the cache immediately, before the server confirms:

```typescript
export function useUpdateOrderStatus() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: ({ orderId, status }: { orderId: string; status: string }) =>
      fetch(`/api/orders/${orderId}`, {
        method: 'PATCH',
        body: JSON.stringify({ status }),
      }).then((r) => r.json()),

    // Before the request completes, optimistically update the cache
    onMutate: async ({ orderId, status }) => {
      await queryClient.cancelQueries({ queryKey: ['orders'] }) // Cancel in-flight refetches

      const previousOrders = queryClient.getQueryData(['orders']) // Save snapshot for rollback

      queryClient.setQueryData(['orders'], (old: Order[]) =>
        old.map((o) => (o.id === orderId ? { ...o, status } : o)) // Optimistic update
      )

      return { previousOrders } // Return context for rollback
    },

    onError: (error, variables, context) => {
      // Rollback on error
      if (context?.previousOrders) {
        queryClient.setQueryData(['orders'], context.previousOrders)
      }
    },

    onSettled: () => {
      // Always refetch to ensure cache is in sync with server
      queryClient.invalidateQueries({ queryKey: ['orders'] })
    },
  })
}
```

---

## SWR — The Lightweight Alternative

**SWR** (Stale-While-Revalidate) by Vercel is a simpler alternative to TanStack Query. It has less features but a smaller API surface:

```typescript
import useSWR from 'swr'

const fetcher = (url: string) => fetch(url).then((r) => r.json())

export default function OrdersPage() {
  const { data: orders, error, isLoading } = useSWR('/api/orders', fetcher, {
    revalidateOnFocus: true,    // Refetch when user returns to tab
    refreshInterval: 30_000,    // Poll every 30 seconds
  })

  if (isLoading) return <p>Loading...</p>
  if (error) return <p>Error</p>

  return <OrderList orders={orders} />
}
```

### TanStack Query vs SWR

| Dimension | TanStack Query | SWR |
| :--- | :--- | :--- |
| **Bundle size** | ~13KB | ~4KB |
| **Mutations** | Full `useMutation` with rollback | Manual |
| **Optimistic updates** | Built-in | Manual |
| **Infinite queries** | Built-in | Built-in |
| **DevTools** | Excellent | Basic |
| **Best for** | Complex apps with mutations | Read-heavy, simple apps |

**Recommendation**: Use TanStack Query. SWR makes mutations significantly harder.

---

## Related Notes
- [[Global state management]]
- [[Local state in React]]
- [[Data fetching MOC]]
- [[REST API integration]]
- [[Optimistic updates]]
- [[State management MOC]]
