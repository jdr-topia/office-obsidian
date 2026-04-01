---
created: 2026-04-01
tags: [concept]
related: [ [[Rendering strategies MOC]], [[Server side rendering (SSR)]], [[Hydration and partial hydration]], [[React server components]], [[Performance requirements]], [[Frontend system design MOC]] ]
status: active
---

# Streaming SSR

## Overview

**Streaming SSR** is an enhancement to traditional SSR where the server doesn't wait for the entire page to be ready before sending a response. Instead, it **streams chunks of HTML to the browser progressively** as each part becomes ready.

In traditional SSR, the server must fetch ALL data, render ALL components, then send the entire HTML at once. If one component's data is slow (e.g., a slow recommendations API that takes 2 seconds), the entire page waits.

With Streaming SSR, React can send the fast parts immediately and stream the slow parts in later.

---

## The Problem With Traditional SSR

```
Traditional SSR Timeline:
──────────────────────────────────────────────────────
  0ms    Request arrives at server
 100ms   Fetch user data (fast)
 200ms   Fetch order summary (fast)
2200ms   Fetch AI recommendations (SLOW — 2 seconds)
         ← Page renders ALL at once
2300ms   Browser receives HTML, paints page
         User waited 2.3 seconds for everything
──────────────────────────────────────────────────────
```

The slow recommendations API held the entire page hostage.

```
Streaming SSR Timeline:
──────────────────────────────────────────────────────
  0ms    Request arrives at server
 100ms   Server sends: HTML shell + user data + order summary
         ← Browser renders partial page IMMEDIATELY
         ← User sees header, order summary, loading spinner for recommendations
 200ms   Fetch AI recommendations... (waiting)
2200ms   Recommendations ready → server streams that chunk
         ← Browser inserts recommendations into the page
──────────────────────────────────────────────────────
```

The user sees useful content at 100ms instead of 2300ms.

---

## How It Works Technically

Streaming uses the **HTTP Transfer-Encoding: chunked** protocol and React's ability to stream HTML in fragments.

Next.js implements this via **React Suspense boundaries**:

1. When React encounters a `<Suspense>` boundary, it immediately renders the `fallback` UI.
2. The HTML for the fallback is streamed to the browser.
3. When the suspended component's data resolves, React streams the real component HTML.
4. The browser replaces the fallback with the real content (without a full page reload or re-render).

---

## Implementing Streaming SSR in Next.js

### Basic Streaming with Suspense

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react'
import OrderSummary from './OrderSummary'         // Fast data (~100ms)
import Recommendations from './Recommendations'   // Slow data (~2000ms)
import UserProfile from './UserProfile'           // Fast data (~50ms)

export default function DashboardPage() {
  return (
    <main>
      {/* These render immediately — no data loading */}
      <h1>Dashboard</h1>

      {/* Streams immediately when data is ready (~50ms) */}
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile />
      </Suspense>

      {/* Streams quickly (~100ms) */}
      <Suspense fallback={<OrderSkeleton />}>
        <OrderSummary />
      </Suspense>

      {/* Renders last (~2000ms) — but doesn't hold up the others */}
      <Suspense fallback={<RecommendationSkeleton />}>
        <Recommendations />
      </Suspense>
    </main>
  )
}
```

```typescript
// app/dashboard/Recommendations.tsx — This is a Server Component
// It fetches its own data — no prop drilling needed

async function getRecommendations() {
  // Simulating a slow API
  const res = await fetch('https://ml-api.example.com/recommendations', {
    cache: 'no-store',
  })
  return res.json()
}

export default async function Recommendations() {
  const recs = await getRecommendations() // This component "suspends" while waiting

  return (
    <section>
      <h2>Recommended for you</h2>
      {recs.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </section>
  )
}
```

### `loading.tsx` — Route-Level Streaming

Next.js has a special `loading.tsx` file convention that automatically wraps the `page.tsx` in a Suspense boundary:

```typescript
// app/dashboard/loading.tsx
// This renders instantly while dashboard/page.tsx is loading

export default function DashboardLoading() {
  return (
    <div>
      <div className="skeleton skeleton-title" />
      <div className="skeleton skeleton-card" />
      <div className="skeleton skeleton-card" />
    </div>
  )
}
```

```
File Structure:
app/
  dashboard/
    loading.tsx  ← Fallback shown instantly
    page.tsx     ← Real content, streamed when ready
```

### Granular Streaming: Multiple Independent Sections

```typescript
// app/product/[id]/page.tsx — Multiple independent data sources
import { Suspense } from 'react'

export default function ProductPage({ params }) {
  return (
    <main>
      {/* Pre-rendered static layout — instant */}
      <Breadcrumb />

      {/* Independent Suspense boundaries = stream independently */}
      <div className="product-layout">
        <Suspense fallback={<ImageGallerySkeleton />}>
          <ProductImages id={params.id} />  {/* Fast: static images */}
        </Suspense>

        <div>
          <Suspense fallback={<DetailsSkeleton />}>
            <ProductDetails id={params.id} />  {/* Medium: DB query */}
          </Suspense>

          <Suspense fallback={<ReviewsSkeleton />}>
            <ProductReviews id={params.id} />  {/* Slow: aggregated data */}
          </Suspense>
        </div>
      </div>
    </main>
  )
}
```

Each Suspense boundary streams independently. Images appear first, then details, then reviews — no component waits for another.

---

## Streaming vs Traditional SSR: Side-by-Side

| Aspect | Traditional SSR | Streaming SSR |
| :--- | :--- | :--- |
| **When page appears** | After ALL data is fetched | After FIRST data is ready |
| **Slow API impact** | Entire page waits | Only that section waits |
| **Implementation** | Default async component | Suspense boundaries |
| **TTFB** | Slow (waits for all data) | Fast (streams partial HTML) |
| **FCP** | Same as TTFB | Much faster |
| **Complexity** | Low | Medium |

---

## Error Handling in Streaming Boundaries

```typescript
// app/dashboard/error.tsx — Co-located error boundary

'use client' // Error boundaries must be Client Components

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log error to monitoring service
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={reset}>Try again</button>
    </div>
  )
}
```

---

## Pros and Cons

### Pros
- **Dramatically faster perceived load time**: Users see content progressively — no waiting for the slowest component.
- **Better UX over traditional SSR**: Loading skeletons instead of blank screens.
- **SEO maintained**: Full HTML is eventually delivered to crawlers.
- **Component-level parallelism**: Each Suspense boundary fetches data independently.

### Cons
- **More complex component architecture**: Requires deliberate placement of Suspense boundaries.
- **TTFB is still dependent on first chunk**: If the outermost component is slow, streaming doesn't help the initial response.
- **Debugging is harder**: Partial renders and async boundaries are harder to trace.
- **Edge cases with layout shift**: If skeleton sizes don't match real content, CLS can still occur.

---

## Related Notes
- [[Server side rendering (SSR)]]
- [[Hydration and partial hydration]]
- [[React server components]]
- [[Performance requirements]]
- [[Core web vitals]]
- [[Rendering strategies MOC]]
