---
created: 2026-04-01
tags: [concept]
related: [ [[Rendering strategies MOC]], [[Static site generation (SSG)]], [[Server side rendering (SSR)]], [[CDN usage in frontend]], [[Stale-while-revalidate]], [[Frontend system design MOC]] ]
status: active
---

# Incremental static regeneration (ISR)

## Overview

**Incremental Static Regeneration (ISR)** is a hybrid rendering model in Next.js that combines the **speed of SSG** with the **data freshness of SSR**. It lets you pre-render static pages at build time, but then **automatically regenerate them in the background** after a specified time interval — without requiring a full site rebuild.

It solves the biggest weakness of SSG: stale data.

> ISR is like a newspaper: printed once and fast to read, but reprinted with fresh news at regular intervals — while you still get the old edition until the new one is ready.

---

## How It Works — The Stale-While-Revalidate Pattern

ISR implements the **stale-while-revalidate** cache strategy:

```
Step 1: BUILD TIME
  → Page /products/iphone is pre-rendered and cached on CDN

Step 2: REQUEST (before revalidation interval)
  User visits /products/iphone
  → CDN serves the cached (stale) HTML immediately
  → User sees fast content (~10ms)

Step 3: REQUEST (after revalidation interval, e.g., 60 seconds)
  User visits /products/iphone
  → CDN still serves the stale cached HTML to THIS user (fast!)
  → Next.js SIMULTANEOUSLY starts regenerating the page in the background
  → New HTML is built with fresh data
  → CDN cache is updated

Step 4: NEXT USER
  → Gets the newly generated page
```

**Key insight**: The user who "triggers" regeneration still gets the old page. The *next* user gets the fresh one. This is the trade-off for zero additional latency.

---

## Implementing ISR in Next.js

### Method 1: Time-Based Revalidation (`revalidate`)

```typescript
// app/products/[id]/page.tsx

async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 60 }, // Revalidate this data every 60 seconds
  })
  return res.json()
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id)

  return (
    <main>
      <h1>{product.name}</h1>
      <p>Price: ${product.price}</p>
      <p>In Stock: {product.stockCount}</p>
    </main>
  )
}
```

You can also set revalidation at the **route segment level**:

```typescript
// app/products/[id]/page.tsx
export const revalidate = 60 // All data fetches in this route revalidate every 60s
```

### Method 2: On-Demand Revalidation (Manual Trigger)

Time-based ISR is good, but sometimes you want to regenerate a page the moment your CMS publishes a change — not after 60 seconds. On-demand revalidation handles this.

```typescript
// app/api/revalidate/route.ts
// Your CMS sends a webhook to this endpoint when content changes

import { NextRequest, NextResponse } from 'next/server'
import { revalidatePath, revalidateTag } from 'next/cache'

export async function POST(request: NextRequest) {
  const secret = request.nextUrl.searchParams.get('secret')

  // Protect the endpoint with a secret
  if (secret !== process.env.REVALIDATION_SECRET) {
    return NextResponse.json({ error: 'Invalid token' }, { status: 401 })
  }

  const body = await request.json()
  const { slug } = body

  // Revalidate the specific product page immediately
  revalidatePath(`/products/${slug}`)

  // OR tag-based revalidation (more flexible)
  revalidateTag('product-data')

  return NextResponse.json({ revalidated: true, path: `/products/${slug}` })
}
```

```typescript
// Then tag your fetches:
async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: {
      tags: ['product-data', `product-${id}`], // Tag this fetch
    },
  })
  return res.json()
}
```

When your CMS publishes a change on product ID 456:
```
POST /api/revalidate?secret=xxx
Body: { "slug": "456" }

→ Next.js immediately regenerates /products/456
→ Next request to that page gets fresh content
```

---

## ISR with `generateStaticParams`

ISR works alongside `generateStaticParams`. Pages in `generateStaticParams` are pre-built. Pages NOT in the list are built on first request (SSR-like) and then cached (ISR-like):

```typescript
// app/products/[id]/page.tsx

// Pre-build the top 100 products at build time
export async function generateStaticParams() {
  const topProducts = await fetch('/api/products?top=100').then((r) => r.json())
  return topProducts.map((p) => ({ id: p.id }))
}

// For product IDs NOT in the top 100:
export const dynamicParams = true // default — SSR on first visit, then ISR cached

export const revalidate = 300 // Revalidate all products every 5 minutes
```

---

## Pros and Cons

### Pros
- **Speed of SSG**: Pre-built pages served from CDN — near-zero TTFB for most requests.
- **Data freshness**: Regular regeneration without full site rebuilds.
- **Scalable**: Can handle massive traffic because most requests hit CDN cache.
- **Excellent SEO**: Full server-rendered HTML on first load.
- **No downtime during update**: Old page served while new one builds in background.

### Cons
- **Stale data window**: A user may see data that's up to `revalidate` seconds old.
- **Not for highly personalized content**: All users see the same cached page.
- **On-demand revalidation requires infrastructure**: Need a webhook endpoint and CMS integration.
- **Complex mental model**: Understanding the stale-while-revalidate cache lifecycle takes practice.

---

## Choosing the Revalidation Interval

| Content Type | Revalidation | Reason |
| :--- | :--- | :--- |
| Blog post | `revalidate = 3600` (1 hour) | Rarely changes |
| Product page | `revalidate = 60` (1 min) | Price/stock changes periodically |
| News headline | `revalidate = 30` (30 sec) | Changes frequently |
| Live sports score | Not ISR — use SSR or WebSocket | Changes every second |
| Marketing page | `revalidate = 86400` (1 day) | Almost never changes |

---

## Related Notes
- [[Static site generation (SSG)]]
- [[Server side rendering (SSR)]]
- [[Stale-while-revalidate]]
- [[CDN usage in frontend]]
- [[Rendering strategies MOC]]
