---
created: 2026-04-01
tags: [concept]
related: [ [[Rendering strategies MOC]], [[SPA vs MPA]], [[Hydration and partial hydration]], [[SEO considerations in frontend design]], [[Server side rendering (SSR)]], [[Frontend system design MOC]] ]
status: active
---

# Client side rendering (CSR)

## Overview

**Client Side Rendering (CSR)** is a rendering model where the server sends a nearly empty HTML shell to the browser, and **JavaScript running in the browser builds the entire UI** by fetching data from APIs and constructing the DOM.

It is the default model of a classic React application (using Vite or `create-react-app`).

---

## How It Works — Step by Step

```
1. User navigates to https://app.example.com/dashboard

2. Browser requests the page from the server
   ← Server responds with:
   <!DOCTYPE html>
   <html>
     <body>
       <div id="root"></div>            ← Empty container
       <script src="/bundle.js"></script> ← App JS
     </body>
   </html>

3. Browser downloads bundle.js (could be 500KB+)

4. Browser parses and executes JavaScript

5. React mounts, calls useEffect / React Query

6. API request: GET /api/dashboard/data

7. Data arrives, React re-renders with real content

8. User FINALLY sees the dashboard
```

**Total time from navigation to visible content: often 2-5 seconds on a slow network.**

---

## When to Use CSR

CSR is the **right choice** when:

| Condition | Reason |
| :--- | :--- |
| Page is behind a login | No SEO needed, no public crawler access |
| Highly interactive UI | Real-time updates, frequent state changes |
| User-specific dynamic data | Data is fully personalized, cannot cache HTML |
| Internal tools / admin panels | Performance from CDN not critical |

**Examples**: Gmail, Notion, Figma, admin dashboards, SaaS app portals.

---

## Implementing CSR in Next.js

In Next.js App Router, CSR is done via `"use client"` components.

### Basic CSR with `useEffect` + `fetch`

```typescript
// app/dashboard/page.tsx
// This entire page renders in the browser

'use client'

import { useEffect, useState } from 'react'

type Order = { id: string; status: string; total: number }

export default function DashboardPage() {
  const [orders, setOrders] = useState<Order[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    async function loadOrders() {
      try {
        const res = await fetch('/api/orders')
        if (!res.ok) throw new Error('Failed to fetch')
        const data = await res.json()
        setOrders(data)
      } catch (err) {
        setError('Could not load orders.')
      } finally {
        setLoading(false)
      }
    }

    loadOrders()
  }, [])

  if (loading) return <p>Loading orders...</p>
  if (error) return <p>{error}</p>

  return (
    <ul>
      {orders.map((order) => (
        <li key={order.id}>
          #{order.id} — {order.status} — ${order.total}
        </li>
      ))}
    </ul>
  )
}
```

### Better CSR with TanStack Query (Recommended)

Using raw `useEffect` for data fetching is fragile. TanStack Query handles caching, loading states, and retries properly:

```typescript
// app/dashboard/page.tsx
'use client'

import { useQuery } from '@tanstack/react-query'

async function fetchOrders() {
  const res = await fetch('/api/orders')
  if (!res.ok) throw new Error('Failed to fetch orders')
  return res.json()
}

export default function DashboardPage() {
  const { data: orders, isLoading, isError } = useQuery({
    queryKey: ['orders'],
    queryFn: fetchOrders,
    staleTime: 30_000, // Consider data fresh for 30 seconds
  })

  if (isLoading) return <p>Loading...</p>
  if (isError) return <p>Something went wrong.</p>

  return (
    <ul>
      {orders.map((order) => (
        <li key={order.id}>{order.status} — ${order.total}</li>
      ))}
    </ul>
  )
}
```

→ See: [[Server state management]] for a full TanStack Query guide.

### Hybrid: SSR Shell + CSR Content (The Common Pattern)

The most common Next.js pattern uses SSR to render the page shell (header, nav, layout) and CSR for dynamic content:

```typescript
// app/dashboard/page.tsx — Server Component (renders shell on server)
import DashboardData from './DashboardData'

export default function DashboardPage() {
  return (
    <main>
      <h1>Dashboard</h1>          {/* Rendered on server */}
      <DashboardData />           {/* This is a Client Component */}
    </main>
  )
}

// app/dashboard/DashboardData.tsx — Client Component
'use client'
import { useQuery } from '@tanstack/react-query'
// ... fetches and renders data client-side
```

---

## Pros and Cons

### Pros
- **Rich interactivity**: Full browser API access — real-time updates, complex animations, drag-and-drop.
- **Reduced server load**: Server only handles API calls, not HTML generation.
- **Fast subsequent navigation**: Once loaded, navigation is instant (SPA-style).
- **Simple mental model**: All rendering logic is in one place — the browser.

### Cons
- **Slow initial page load**: Must download, parse, and execute JS before anything is visible.
- **Poor SEO**: Search engine crawlers see an empty HTML shell (without SSR/prerendering).
- **Worse performance on low-end devices**: Parsing heavy JS bundles is CPU-intensive.
- **Layout shift**: UI flashes from loading → data state, causing CLS.
- **Waterfalling**: Sequential fetches (fetch user → fetch orders based on user) add to load time.

---

## Performance Pitfalls and How to Fix Them

| Pitfall | Impact | Fix in Next.js |
| :--- | :--- | :--- |
| Rendering entire page as Client Component | Large JS bundle | Use Server Components for static parts |
| Fetching in series (waterfall) | Slow load | Use `Promise.all` or React Query parallel queries |
| No loading skeleton | High CLS | Use `loading.tsx` Suspense fallback |
| Over-fetching data | Wasted bandwidth | Use field selection (GraphQL) or BFF data shaping |

---

## CSR vs SSR Mental Model

> **Use CSR when the content is for the logged-in user and doesn't need to be seen by Google.**
> **Use SSR/SSG when the content needs to be crawled or when the first paint needs to be fast for all users.**

---

## Related Notes
- [[Server side rendering (SSR)]]
- [[Static site generation (SSG)]]
- [[Hydration and partial hydration]]
- [[Server state management]]
- [[SPA vs MPA]]
- [[Rendering strategies MOC]]
