---
created: 2026-04-01
tags: [concept, how-to]
related: [ [[Rendering strategies MOC]], [[SPA vs MPA]], [[SEO considerations in frontend design]], [[Frontend system design MOC]], [[React server components]] ]
status: active
---


- [[#1. Server-Side Rendering (SSR)|1. Server-Side Rendering (SSR)]]
	- [[#1. Server-Side Rendering (SSR)#What it is|What it is]]
	- [[#1. Server-Side Rendering (SSR)#How it works|How it works]]
	- [[#1. Server-Side Rendering (SSR)#Example|Example]]
	- [[#1. Server-Side Rendering (SSR)#Use Cases|Use Cases]]
	- [[#1. Server-Side Rendering (SSR)#Pros|Pros]]
	- [[#1. Server-Side Rendering (SSR)#Cons|Cons]]
- [[#2. Static Site Generation (SSG)|2. Static Site Generation (SSG)]]
	- [[#2. Static Site Generation (SSG)#What it is|What it is]]
	- [[#2. Static Site Generation (SSG)#How it works|How it works]]
	- [[#2. Static Site Generation (SSG)#Example|Example]]
	- [[#2. Static Site Generation (SSG)#Use Cases|Use Cases]]
	- [[#2. Static Site Generation (SSG)#Pros|Pros]]
	- [[#2. Static Site Generation (SSG)#Cons|Cons]]
- [[#3. Incremental Static Regeneration (ISR)|3. Incremental Static Regeneration (ISR)]]
	- [[#3. Incremental Static Regeneration (ISR)#What it is|What it is]]
	- [[#3. Incremental Static Regeneration (ISR)#Example|Example]]
	- [[#3. Incremental Static Regeneration (ISR)#How it works|How it works]]
	- [[#3. Incremental Static Regeneration (ISR)#Use Cases|Use Cases]]
	- [[#3. Incremental Static Regeneration (ISR)#Pros|Pros]]
	- [[#3. Incremental Static Regeneration (ISR)#Cons|Cons]]
- [[#4. Client-Side Rendering (CSR)|4. Client-Side Rendering (CSR)]]
	- [[#4. Client-Side Rendering (CSR)#What it is|What it is]]
	- [[#4. Client-Side Rendering (CSR)#Example|Example]]
	- [[#4. Client-Side Rendering (CSR)#Use Cases|Use Cases]]
	- [[#4. Client-Side Rendering (CSR)#Pros|Pros]]
	- [[#4. Client-Side Rendering (CSR)#Cons|Cons]]
- [[#5. Streaming (React Server Components + Suspense)|5. Streaming (React Server Components + Suspense)]]
	- [[#5. Streaming (React Server Components + Suspense)#What it is|What it is]]
	- [[#5. Streaming (React Server Components + Suspense)#Example|Example]]
	- [[#5. Streaming (React Server Components + Suspense)#Use Cases|Use Cases]]
	- [[#5. Streaming (React Server Components + Suspense)#Pros|Pros]]
	- [[#5. Streaming (React Server Components + Suspense)#Cons|Cons]]
- [[#6. Edge Rendering|6. Edge Rendering]]
	- [[#6. Edge Rendering#What it is|What it is]]
	- [[#6. Edge Rendering#Example|Example]]
	- [[#6. Edge Rendering#Use Cases|Use Cases]]
	- [[#6. Edge Rendering#Pros|Pros]]
	- [[#6. Edge Rendering#Cons|Cons]]
- [[#7. Partial Prerendering (PPR) (New)|7. Partial Prerendering (PPR) (New)]]
	- [[#7. Partial Prerendering (PPR) (New)#What it is|What it is]]
	- [[#7. Partial Prerendering (PPR) (New)#Concept|Concept]]
	- [[#7. Partial Prerendering (PPR) (New)#Use Cases|Use Cases]]
	- [[#7. Partial Prerendering (PPR) (New)#Pros|Pros]]
	- [[#7. Partial Prerendering (PPR) (New)#Cons|Cons]]


Next.js provides multiple rendering strategies to balance **performance, SEO, freshness, and scalability**.

Understanding _when to use what_ is more important than just knowing _how it works_.

---

## 1. Server-Side Rendering (SSR)

### What it is

HTML is generated **on every request** on the server.

### How it works

- User hits request
    
- Server fetches data
    
- Server returns fully rendered HTML
    

### Example

```tsx
// app/products/[id]/page.tsx (App Router - SSR by default with dynamic data)

async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    cache: "no-store", // forces SSR
  });
  return res.json();
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);

  return <div>{product.name}</div>;
}
```

---

### Use Cases

- User-specific data (dashboard, profile)
    
- Frequently changing data
    
- Auth-based content
    
- Real-time-ish systems
    

---

### Pros

- Always fresh data
    
- Good SEO
    
- Secure (logic runs on server)
    

---

### Cons

- Slower response (server work per request)
    
- Higher server cost
    
- Can hurt performance under load
    

---

## 2. Static Site Generation (SSG)

### What it is

HTML is generated **at build time**.

### How it works

- During build → pages are pre-rendered
    
- Served instantly via CDN
    

---

### Example

```tsx
// app/blog/[slug]/page.tsx

async function getPost(slug: string) {
  const res = await fetch(`https://api.example.com/posts/${slug}`);
  return res.json();
}

export async function generateStaticParams() {
  return [{ slug: "post-1" }, { slug: "post-2" }];
}

export default async function BlogPage({ params }) {
  const post = await getPost(params.slug);

  return <div>{post.title}</div>;
}
```

---

### Use Cases

- Blogs
    
- Marketing pages
    
- Documentation
    
- Landing pages
    

---

### Pros

- Extremely fast (served from CDN)
    
- Great SEO
    
- No server cost per request
    

---

### Cons

- Data becomes stale
    
- Rebuild required for updates
    
- Not suitable for dynamic data
    

---

## 3. Incremental Static Regeneration (ISR)

### What it is

Hybrid of **SSG + SSR**

- Pre-rendered page
    
- Automatically re-generated **after a time interval**
    

---

### Example

```tsx
async function getProducts() {
  const res = await fetch("https://api.example.com/products", {
    next: { revalidate: 60 }, // revalidate every 60 seconds
  });
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <div>
      {products.map(p => (
        <div key={p.id}>{p.name}</div>
      ))}
    </div>
  );
}
```

---

### How it works

1. First request → serves static page
    
2. After 60s → next request triggers background regeneration
    
3. Old page served until new one is ready
    

---

### Use Cases

- E-commerce product pages
    
- News websites
    
- Content that updates periodically
    

---

### Pros

- Fast like SSG
    
- Data stays reasonably fresh
    
- No full rebuild needed
    

---

### Cons

- Slightly stale data window
    
- More complex mental model
    
- Cache invalidation complexity
    

---

## 4. Client-Side Rendering (CSR)

### What it is

Rendering happens **in the browser using JavaScript**

---

### Example

```tsx
"use client";

import { useEffect, useState } from "react";

export default function Products() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch("/api/products")
      .then(res => res.json())
      .then(setProducts);
  }, []);

  return (
    <div>
      {products.map(p => (
        <div key={p.id}>{p.name}</div>
      ))}
    </div>
  );
}
```

---

### Use Cases

- Dashboards
    
- Admin panels
    
- Highly interactive UI
    
- Non-SEO pages
    

---

### Pros

- Fast interactions after load
    
- Reduces server load
    
- Great for dynamic UI
    

---

### Cons

- Poor SEO
    
- Slower initial load
    
- Depends on JS
    

---

## 5. Streaming (React Server Components + Suspense)

### What it is

Send HTML **in chunks** instead of waiting for full page.

---

### Example

```tsx
import { Suspense } from "react";

function ProductList() {
  // slow API
}

export default function Page() {
  return (
    <div>
      <h1>Products</h1>

      <Suspense fallback={<p>Loading...</p>}>
        <ProductList />
      </Suspense>
    </div>
  );
}
```

---

### Use Cases

- Large pages with multiple data sources
    
- Slow APIs
    
- Improving perceived performance
    

---

### Pros

- Faster perceived load
    
- Better UX
    
- Partial rendering
    

---

### Cons

- More complex
    
- Requires proper boundaries
    
- Debugging can be tricky
    

---

## 6. Edge Rendering

### What it is

Run SSR closer to user via **Edge Functions**

---

### Example

```tsx
export const runtime = "edge";

export default async function Page() {
  const res = await fetch("https://api.example.com/data");
  const data = await res.json();

  return <div>{data.name}</div>;
}
```

---

### Use Cases

- Geo-based personalization
    
- Low-latency SSR
    
- Middleware logic
    

---

### Pros

- Ultra-fast global response
    
- Reduced latency
    
- Scalable
    

---

### Cons

- Limited Node.js APIs
    
- Cold starts (sometimes)
    
- Debugging harder
    

---

## 7. Partial Prerendering (PPR) (New)

### What it is

Mix of:

- Static shell (fast)
    
- Dynamic parts streamed later
    

---

### Concept

```txt
[Static Layout] → loads instantly
[Dynamic Content] → loads later via streaming
```

---

### Use Cases

- Complex pages (Amazon-like)
    
- Mix of static + dynamic content
    

---

### Pros

- Best of both worlds
    
- Great UX
    
- Fast + dynamic
    

---

### Cons

- Still evolving
    
- Requires careful architecture
    

---

# Quick Comparison Table

|Strategy|Speed|SEO|Fresh Data|Server Cost|
|---|---|---|---|---|
|SSR|Medium|High|High|High|
|SSG|Very Fast|High|Low|Low|
|ISR|Fast|High|Medium|Low|
|CSR|Slow initial|Low|High|Low|
|Streaming|Fast perceived|High|High|Medium|
|Edge|Very Fast|High|High|Medium|

---

# Decision Guide (Real-world Thinking)

- **Need SEO + real-time data?** → SSR
    
- **Static content?** → SSG
    
- **Mostly static but updates?** → ISR
    
- **Highly interactive app?** → CSR
    
- **Slow APIs?** → Streaming
    
- **Global low latency?** → Edge
    
- **Complex hybrid UI?** → PPR
    

---

# Key Insight (Important)

> In modern Next.js (App Router), rendering is **not page-level anymore** — it's **component-level**

You can mix:

- SSR + CSR + Streaming in the same page
    

---

# Common Mistakes

- Using SSR for everything → costly
    
- Using CSR for SEO pages → bad ranking
    
- Not using ISR where applicable → unnecessary rebuilds
    
- Over-fetching on client instead of server
    

---

# Mental Model

Think like this:

```txt
Where should this data be rendered?

Server → secure, SEO, heavy logic
Build time → static, fast
Client → interactive
Edge → fast global logic
```

---

## Related Notes
- [[Rendering strategies MOC]]
- [[SPA vs MPA]]
- [[SEO considerations in frontend design]]
- [[React server components]]
- [[Hydration and partial hydration]]
- [[Islands architecture]]
- [[Frontend system design MOC]]
