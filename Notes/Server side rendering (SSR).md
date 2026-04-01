---
created: 2026-04-01
tags: [concept]
related: [ [[Rendering strategies MOC]], [[Client side rendering (CSR)]], [[Static site generation (SSG)]], [[Hydration and partial hydration]], [[React server components]], [[SEO considerations in frontend design]], [[Frontend system design MOC]] ]
status: active
---

# Server side rendering (SSR)

## Overview

**Server Side Rendering (SSR)** is a rendering model where the server generates the complete HTML for a page **on every incoming request**, before sending it to the browser.

The user receives a fully-formed HTML document — the browser can paint content immediately, without waiting for JavaScript to download and execute.

SSR solves the two biggest problems with CSR: **slow initial load** and **poor SEO**.

---

## How It Works — Step by Step

```
1. User navigates to https://shop.example.com/products/iphone-15

2. Browser sends GET /products/iphone-15 to the Next.js server

3. Next.js server:
   a. Receives the request
   b. Fetches product data from DB/API (server-side)
   c. Renders the React component tree to HTML
   d. Sends the fully-rendered HTML to the browser

4. Browser receives:
   <!DOCTYPE html>
   <html>
     <body>
       <h1>iPhone 15 Pro</h1>       ← Real content! Not a div#root
       <p>Starting from $999...</p>
       <img src="/iphone15.jpg" />
     </body>
   </html>

5. Browser paints content immediately (fast LCP)

6. Next.js hydrates the page — attaches React event listeners
   → Page becomes fully interactive

Total time to first content: ~300-600ms (just server + network)
```

---

## Implementing SSR in Next.js App Router

In the App Router, **Server Components are SSR by default**. You don't need any special flags — just `async` components that fetch data.

### Basic SSR Page

```typescript
// app/products/[id]/page.tsx
// This is a Server Component — runs on the server per request

type Product = {
  id: string
  name: string
  price: number
  description: string
  imageUrl: string
}

async function getProduct(id: string): Promise<Product> {
  // This fetch runs on the SERVER — database credentials are never exposed to browser
  const res = await fetch(`https://api.example.com/products/${id}`, {
    cache: 'no-store', // Opt out of caching → SSR (re-fetch on every request)
  })

  if (!res.ok) throw new Error('Product not found')
  return res.json()
}

export default async function ProductPage({
  params,
}: {
  params: { id: string }
}) {
  const product = await getProduct(params.id)

  return (
    <main>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>Price: ${product.price}</p>
    </main>
  )
}
```

> `cache: 'no-store'` is the key that opts out of Next.js's data cache and forces a fresh data fetch per request — this is what makes it SSR.

### SSR with Dynamic Metadata

```typescript
// app/products/[id]/page.tsx
import { Metadata } from 'next'

// generateMetadata also runs on the server per request
export async function generateMetadata({
  params,
}: {
  params: { id: string }
}): Promise<Metadata> {
  const product = await getProduct(params.id)

  return {
    title: `${product.name} | MyStore`,
    description: product.description,
    openGraph: { images: [product.imageUrl] },
  }
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id)
  return <ProductDetails product={product} />
}
```

### SSR with Multiple Parallel Data Sources

```typescript
// Fetching multiple data sources in parallel — avoids waterfall
export default async function OrderDetailPage({ params }) {
  // Promise.all runs all fetches concurrently on the server
  const [order, customer, products] = await Promise.all([
    getOrder(params.orderId),
    getCustomer(params.customerId),
    getProductsInOrder(params.orderId),
  ])

  return (
    <div>
      <h1>Order #{order.id}</h1>
      <p>Customer: {customer.name}</p>
      <ProductList products={products} />
    </div>
  )
}
```

### SSR with Authentication

```typescript
// app/dashboard/page.tsx — Protected SSR page
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const session = await getServerSession(authOptions)

  if (!session) {
    redirect('/login') // Server-side redirect — no flash of unauthenticated content
  }

  const data = await fetchDashboardData(session.user.id)

  return <Dashboard data={data} user={session.user} />
}
```

---

## SSR vs Server Components: Are They the Same?

This is a common confusion point in Next.js.

| | Classic SSR (Pages Router) | React Server Components (App Router) |
| :--- | :--- | :--- |
| **When generated** | Per request on the server | Per request on the server |
| **JS sent to browser** | Full component JS | **Zero** — Server Component code never ships to browser |
| **Hydration** | Full page hydration | Selective (only Client Components hydrate) |
| **Data fetching** | `getServerSideProps` | `async` component body directly |

React Server Components (App Router) are an evolution of SSR — they are more efficient because **Server Component JavaScript is never included in the client bundle**.

→ See: [[React server components]] for the full breakdown.

---

## Pros and Cons

### Pros
- **Excellent SEO**: Full HTML is available immediately for crawlers.
- **Fast First Contentful Paint (FCP)**: Browser paints real content before JS loads.
- **Secure**: Sensitive logic (DB queries, API keys) runs on the server.
- **Personalized content with SEO**: Unlike SSG, can return user-specific HTML (e.g., `Hello, Raman` on the page).

### Cons
- **Slower Time to First Byte (TTFB)**: Server must complete the render before sending response. SSG is faster.
- **Higher server cost**: Every request runs server-side code. Under high traffic, this adds up.
- **Cannot be fully served from CDN**: Dynamic, per-request HTML cannot be edge-cached (unlike SSG).
- **Hydration cost**: After receiving HTML, the browser still runs JS to "hydrate" and attach event listeners.

---

## When to Use SSR

| Scenario | Use SSR? | Why |
| :--- | :--- | :--- |
| Product detail page (changes hourly) | ✅ Yes | Data must be fresh, SEO needed |
| User dashboard (personalized) | ✅ Yes | User-specific, cannot be pre-cached |
| Blog post (doesn't change) | ❌ No, use SSG | Static content — no benefit from SSR |
| Admin panel | ❌ No, use CSR | Behind login, SEO irrelevant |
| Product listing (updates daily) | ⚠️ Use ISR | Mostly static, doesn't need per-request render |

---

## Related Notes
- [[Client side rendering (CSR)]]
- [[Static site generation (SSG)]]
- [[Incremental static regeneration (ISR)]]
- [[React server components]]
- [[Hydration and partial hydration]]
- [[Streaming SSR]]
- [[Rendering strategies MOC]]
