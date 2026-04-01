---
created: 2026-04-01
tags: [concept]
related: [ [[Rendering strategies MOC]], [[Server side rendering (SSR)]], [[Incremental static regeneration (ISR)]], [[CDN usage in frontend]], [[SEO considerations in frontend design]], [[Frontend system design MOC]] ]
status: active
---

# Static site generation (SSG)

## Overview

**Static Site Generation (SSG)** is a rendering model where HTML pages are generated **once at build time** — not on every request. The output is a set of static `.html` files that can be uploaded to a CDN and served at near-zero latency to users worldwide.

SSG is the **fastest possible way** to serve a web page. There is no server to execute, no database to query, no rendering to perform at request time. The browser simply downloads a pre-built file.

> Think of SSG as "baking the cake before customers arrive" vs. SSR's "baking the cake only when someone orders it."

---

## How It Works — Step by Step

```
BUILD TIME (runs once, e.g., during CI/CD deploy):
  Next.js build process starts
    ↓
  Fetches data from CMS / API / DB
    ↓
  Renders all React components to HTML files
    ↓
  Outputs: /about.html, /blog/post-1.html, /products/iphone.html
    ↓
  Files uploaded to CDN (e.g., Vercel, CloudFront)

REQUEST TIME (runs for every user, from CDN):
  User visits https://example.com/blog/post-1
    ↓
  CDN edge node (geographically close) serves pre-built HTML
    ↓
  Ultra-fast response (~10-50ms TTFB)
    ↓
  Browser paints content immediately
    ↓
  Next.js hydrates → page becomes interactive
```

---

## Implementing SSG in Next.js App Router

In Next.js App Router, SSG is the default behaviour when data fetching uses the default cache settings.

### Static Page (No Data)

```typescript
// app/about/page.tsx
// No data fetching → statically rendered at build time by default

export default function AboutPage() {
  return (
    <main>
      <h1>About Us</h1>
      <p>We are a software company...</p>
    </main>
  )
}
```

### SSG with Data Fetching

```typescript
// app/blog/page.tsx
// Fetching without cache: 'no-store' → Next.js caches this at build time → SSG

async function getPosts() {
  const res = await fetch('https://api.example.com/posts')
  // Default fetch behaviour in Next.js = cached → SSG
  return res.json()
}

export default async function BlogListPage() {
  const posts = await getPosts()

  return (
    <main>
      <h1>Blog</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <a href={`/blog/${post.slug}`}>{post.title}</a>
          </li>
        ))}
      </ul>
    </main>
  )
}
```

### SSG with Dynamic Routes (`generateStaticParams`)

For pages like `/blog/[slug]`, you must tell Next.js which slugs to pre-build at build time:

```typescript
// app/blog/[slug]/page.tsx

type Post = { slug: string; title: string; content: string; publishedAt: string }

// Tell Next.js which paths to generate at build time
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then((r) => r.json())

  // Returns an array of params objects
  return posts.map((post: Post) => ({
    slug: post.slug,
  }))
  // Next.js will now build /blog/intro-to-react, /blog/nextjs-guide, etc.
}

async function getPost(slug: string): Promise<Post> {
  const res = await fetch(`https://api.example.com/posts/${slug}`)
  if (!res.ok) throw new Error('Post not found')
  return res.json()
}

export default async function BlogPostPage({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)

  return (
    <article>
      <h1>{post.title}</h1>
      <time>{post.publishedAt}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  )
}
```

### SSG + Dynamic Metadata

```typescript
// Metadata is also generated at build time for SSG pages
export async function generateMetadata({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  return {
    title: `${post.title} | My Blog`,
    description: post.excerpt,
  }
}
```

---

## Handling Paths Not in `generateStaticParams`

What happens when a user visits `/blog/a-new-post` that wasn't in the build?

```typescript
// app/blog/[slug]/page.tsx

// Option 1: Return 404
export const dynamicParams = false // Any path not in generateStaticParams → 404

// Option 2: Fall back to SSR for unknown paths (dynamicParams = true is the default)
export const dynamicParams = true
// Unknown paths will be server-rendered on first request,
// then cached for subsequent requests (On-Demand ISR)
```

---

## Pros and Cons

### Pros
- **Fastest possible TTFB**: Pre-built HTML served directly from CDN.
- **Excellent SEO**: Full HTML available on first load with no JS required.
- **Zero server cost per request**: No compute happens at request time.
- **Highly scalable**: CDN can handle virtually unlimited traffic.
- **Secure**: No server execution = reduced attack surface.

### Cons
- **Stale data**: Content is frozen at build time. A new blog post only appears after a rebuild.
- **Long build times at scale**: 10,000 product pages = 10,000 pages to build. Can take minutes.
- **Not suitable for personalized content**: Same HTML for every user — can't say "Hello, Raman."
- **Rebuild required for updates**: Every content change requires a new deployment.

---

## When to Use SSG

| Content Type | SSG? | Why |
| :--- | :--- | :--- |
| Marketing / landing pages | ✅ Yes | Never changes, max performance |
| Blog posts / documentation | ✅ Yes | Static content, great SEO |
| E-commerce product pages | ⚠️ Use ISR | Prices/stock change — ISR is better |
| User dashboards | ❌ No | Personalized, use SSR or CSR |
| News article (updated hourly) | ⚠️ Use ISR | SSG gets stale too fast |

---

## SSG vs ISR

SSG is a subset of ISR. The difference:

| | SSG | ISR |
| :--- | :--- | :--- |
| **When generated** | Once at build | At build + periodically in background |
| **Data freshness** | Stale until redeploy | Configurable (e.g., fresh every 60s) |
| **Rebuilds needed** | Yes, for updates | No — background regeneration |

→ See: [[Incremental static regeneration (ISR)]]

---

## Related Notes
- [[Incremental static regeneration (ISR)]]
- [[Server side rendering (SSR)]]
- [[CDN usage in frontend]]
- [[SEO considerations in frontend design]]
- [[Hydration and partial hydration]]
- [[Rendering strategies MOC]]
