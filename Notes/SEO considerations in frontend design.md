---
created: 2026-04-01
tags: [concept]
related: [ [[Requirement analysis in frontend design]], [[Functional vs non-functional requirements]], [[Client side rendering (CSR)]], [[Server side rendering (SSR)]], [[Static site generation (SSG)]], [[Frontend system design MOC]] ]
status: active
---

# SEO considerations in frontend design

## Overview

SEO (Search Engine Optimization) in the context of frontend design is about ensuring your application's content can be **discovered, crawled, and indexed by search engines** like Google. The rendering architecture you choose has a massive impact on your SEO.

This is a critical requirement for any public-facing product — marketing sites, e-commerce, news platforms, landing pages, documentation.

> **Internal dashboards or gated apps** (behind login) generally do not need SEO consideration at all.

---

## Why Frontend Architecture Directly Affects SEO

The fundamental problem: **search engine crawlers need to read your HTML**.

### The Problem with Client Side Rendering (CSR)

In a pure React SPA (CSR), the initial HTML the server sends is nearly empty:

```html
<!-- What Googlebot receives from a pure CSR app -->
<!DOCTYPE html>
<html>
  <head><title>My App</title></head>
  <body>
    <div id="root"></div>  <!-- Empty! React hasn't run yet -->
    <script src="/bundle.js"></script>
  </body>
</html>
```

Google's crawler must then:
1. Download and execute your JS bundle.
2. Wait for React to render.
3. Index the content.

Problems: slow, unreliable, and some content may never be indexed at all.

### The Solution: Server-Rendered HTML

With SSR (Next.js) or SSG, the server sends fully-rendered HTML:

```html
<!-- What Googlebot receives from a Next.js SSR/SSG app -->
<!DOCTYPE html>
<html>
  <head>
    <title>Product: iPhone 15 Pro | MyStore</title>
    <meta name="description" content="Buy the iPhone 15 Pro at the best price..." />
  </head>
  <body>
    <main>
      <h1>iPhone 15 Pro</h1>
      <p>Starting from $999...</p>
      <!-- Full content rendered! -->
    </main>
  </body>
</html>
```

The crawler reads real content immediately. No JS execution required.

→ See full comparison: [[Client side rendering (CSR)]] and [[Server side rendering (SSR)]]

---

## SEO Requirements Checklist

### Metadata
- [ ] Every page has a unique `<title>` tag (55-60 characters ideally).
- [ ] Every page has a unique `<meta name="description">` (150-160 characters).
- [ ] Open Graph tags for social sharing (`og:title`, `og:image`, `og:description`).

**Implementing in Next.js App Router:**

```typescript
// app/products/[id]/page.tsx
import type { Metadata } from 'next'

// Static metadata
export const metadata: Metadata = {
  title: 'iPhone 15 Pro | MyStore',
  description: 'Buy the iPhone 15 Pro at the best price. Free shipping available.',
  openGraph: {
    title: 'iPhone 15 Pro | MyStore',
    description: 'Buy the iPhone 15 Pro at the best price.',
    images: ['/images/iphone15pro.jpg'],
  },
}

// Dynamic metadata (for dynamic routes)
export async function generateMetadata({ params }): Promise<Metadata> {
  const product = await fetchProduct(params.id)

  return {
    title: `${product.name} | MyStore`,
    description: product.seoDescription,
    openGraph: {
      images: [product.imageUrl],
    },
  }
}
```

### URL Structure
- [ ] URLs are clean and descriptive (e.g., `/blog/react-server-components` not `/blog?id=123`).
- [ ] No duplicate content on different URLs.
- [ ] Canonical tags where needed.

```typescript
// Setting canonical URL in Next.js
export const metadata: Metadata = {
  alternates: {
    canonical: 'https://example.com/products/iphone-15-pro',
  },
}
```

### Structured Data (Schema.org)
Helps Google understand the type of content on your page, enabling rich snippets.

```typescript
// app/products/[id]/page.tsx
export default function ProductPage({ product }) {
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    offers: {
      '@type': 'Offer',
      price: product.price,
      priceCurrency: 'USD',
    },
  }

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
      />
      <h1>{product.name}</h1>
      {/* ... */}
    </>
  )
}
```

### Sitemap
```typescript
// app/sitemap.ts — Next.js auto-generates sitemap at /sitemap.xml
import type { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: 'https://example.com', lastModified: new Date() },
    { url: 'https://example.com/products', lastModified: new Date() },
  ]
}
```

### robots.txt
```typescript
// app/robots.ts
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/' },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

---

## When SEO Is NOT a Requirement

| App Type | Needs SEO? | Preferred Rendering |
| :--- | :--- | :--- |
| Marketing site | Yes | SSG |
| E-commerce | Yes | SSG + ISR or SSR |
| Blog/News | Yes | SSG + ISR |
| Admin dashboard | No | CSR is fine |
| Customer portal (gated) | No | CSR is fine |

---

## Related Notes
- [[Client side rendering (CSR)]]
- [[Server side rendering (SSR)]]
- [[Static site generation (SSG)]]
- [[Incremental static regeneration (ISR)]]
- [[Requirement analysis in frontend design]]
