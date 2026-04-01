---
created: 2026-04-01
tags: [concept]
related: [ [[High level architecture MOC]], [[Backend for Frontend (BFF)]], [[Performance requirements]], [[Image and font optimization]], [[Frontend system design MOC]] ]
status: active
---

# CDN usage in frontend

## Overview

A **CDN (Content Delivery Network)** is a globally distributed network of servers that cache and serve your static assets (JS, CSS, images, fonts) from locations geographically close to your users.

Without a CDN, every user in India downloading your JavaScript bundle would make a request all the way to your server in the US — adding 150-300ms of network latency on every single asset.

With a CDN, that same user downloads from a CDN server in Mumbai, reducing latency to < 10ms.

---

## How a CDN Works

```
Without CDN:
  User (Mumbai) → Server (US East) → ~250ms latency
  User (Tokyo)  → Server (US East) → ~180ms latency

With CDN:
  User (Mumbai) → CDN Edge Node (Mumbai) → ~8ms latency
  User (Tokyo)  → CDN Edge Node (Tokyo)  → ~5ms latency

CDN Edge Node (Mumbai) → Origin Server (US East) → Only on MISS
```

**Cache Miss**: CDN doesn't have the file yet → fetches from your origin server → caches it → serves user.
**Cache Hit**: CDN has the file → serves user directly. No origin server call.

---

## What to Put on a CDN

### Always put on CDN:
- JavaScript bundles (`/static/js/main.abc123.js`)
- CSS files (`/static/css/main.abc123.css`)
- Images (`/images/hero.webp`)
- Fonts (`/fonts/Inter-Regular.woff2`)
- Favicons and manifest files

### Conditionally put on CDN:
- HTML pages (for SSG or ISR pages — Next.js + Vercel does this automatically)
- API responses (for public, non-personalized responses with cache headers)

### Never put on CDN:
- User-specific API responses (private, personalized data)
- Authenticated endpoints
- File uploads (upload directly to object storage like S3)

---

## CDN in Next.js

### Automatic CDN via Vercel

When you deploy to Vercel, **all static assets are served via Vercel's global Edge Network automatically**. No configuration needed:

```typescript
// next.config.js — Vercel handles CDN automatically
// Your /public folder and all built assets are served from Edge

// Images go through next/image which uses Vercel's Image Optimization CDN
import Image from 'next/image'
export default function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={1200}
      height={600}
      priority // Preloads — critical for LCP
    />
  )
}
```

### Custom CDN (e.g., Cloudflare, AWS CloudFront)

If self-hosting, configure an `assetPrefix` to serve static assets from your CDN:

```javascript
// next.config.js
module.exports = {
  assetPrefix: process.env.NODE_ENV === 'production'
    ? 'https://cdn.example.com'
    : '',
}
```

Now all Next.js static files (`_next/static/...`) will be requested from `https://cdn.example.com/_next/static/...`.

---

## Cache Control Headers

The CDN respects your `Cache-Control` headers to know how long to cache files.

```typescript
// app/api/public-data/route.ts
// Public data that can be cached for 1 hour on CDN:
export async function GET() {
  const data = await fetchPublicData()
  return Response.json(data, {
    headers: {
      'Cache-Control': 'public, max-age=3600, stale-while-revalidate=86400',
      // public: CDN can cache this
      // max-age=3600: fresh for 1 hour
      // stale-while-revalidate=86400: serve stale while refreshing, up to 24h
    },
  })
}

// Private data — must NOT be cached by CDN:
export async function GET(req) {
  const userData = await fetchUserData()
  return Response.json(userData, {
    headers: {
      'Cache-Control': 'private, no-store',
      // private: only browser can cache, never CDN
    },
  })
}
```

---

## Asset Hashing: Enabling Long-Term Caching

Static assets (JS/CSS) from a build have **content hashes** in their filenames:
```
main.abc123de.js   ← hash changes when file content changes
```

This enables "cache forever" headers because:
- If the file hasn't changed, the URL is the same → CDN cache hit → 0 network requests.
- If the file has changed, the URL is different → CDN cache miss → downloads new file.

```javascript
// next.config.js — Next.js does this automatically
// All built assets have content hashes by default
// Cache-Control: public, max-age=31536000, immutable
//  ↑ Set by Next.js automatically for the _next/static/ directory
```

---

## CDN for Edge Rendering

Modern CDNs (Vercel Edge, Cloudflare Workers) can also **execute your server code** at the edge, not just serve static files:

```typescript
// app/api/geo-banner/route.ts
export const runtime = 'edge' // Run at CDN edge, not origin server

export async function GET(request: Request) {
  const country = request.headers.get('x-vercel-ip-country') || 'US'

  return Response.json({
    message: country === 'IN' ? 'Free delivery in India!' : 'Free delivery $50+',
  })
}
```

This executes in Mumbai for Indian users and in Tokyo for Japanese users — no round trip to the origin.

---

## Related Notes
- [[High level architecture MOC]]
- [[Performance requirements]]
- [[Image and font optimization]]
- [[HTTP caching and prefetching]]
- [[SEO considerations in frontend design]]
