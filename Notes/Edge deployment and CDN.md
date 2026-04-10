---
created: 2026-04-10
tags: [build, cdn, edge, deployment, performance]
related: [ [[Build and deployment MOC]], [[CI/CD for frontend]], [[Network optimization patterns]] ]
status: active
---

# Edge deployment and CDN

Edge deployment means running your application's compute and serving its assets from servers geographically distributed around the world — as close to the user as possible. It reduces latency by eliminating the physical distance between the user and the server.

---

## CDN (Content Delivery Network)

A CDN caches static assets (JS, CSS, images, fonts) at edge nodes globally. When a user requests a file, the CDN serves it from the nearest node instead of the origin server.

```
User in Mumbai → CDN edge in Mumbai (5ms) vs Origin server in US-East (200ms)
```

### How Caching Works on a CDN

```
1. User requests /assets/main.a3f9b2.js
2. CDN checks: do I have this file?
   - Yes → Serve immediately (cache HIT)
   - No  → Fetch from origin, cache it, serve it (cache MISS)
3. Future requests for same file = instant cache HIT
```

**Cache busting via content hashing**: When you deploy, Vite/Next.js renames `main.js` to `main.a3f9b2.js` (where the hash reflects the file content). If the content changes, the hash changes, and the old cached version is naturally replaced.

---

## Edge Functions

Edge Functions are server-side JavaScript/TypeScript code that runs at the CDN edge, not at the origin. They execute in milliseconds because:
- No cold starts (always warm).
- Running in the same region as the user.
- Use lightweight runtimes (V8 isolates, not Node.js).

### Use Cases

| Use Case | Why Edge? |
|---|---|
| **Authentication / Middleware** | Check session cookie and redirect before the page loads |
| **A/B testing / Feature flags** | Rewrite the URL or modify the response at the edge |
| **Geolocation-based routing** | Forward users to region-specific content |
| **Rate limiting** | Block abusive traffic before it hits the origin |
| **Personalization** | Modify HTML based on cookies/headers at the edge |

### Next.js Middleware (Runs at the Edge)

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token');

  // Redirect unauthenticated users before the page even renders
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Geo-based routing
  const country = request.geo?.country ?? 'US';
  if (country === 'DE') {
    return NextResponse.rewrite(new URL('/de', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/'],
};
```

---

## Vercel vs Cloudflare vs AWS CloudFront

| Feature | Vercel | Cloudflare Pages | AWS CloudFront |
|---|---|---|---|
| **DX / Setup** | ⭐ Easiest | Easy | Complex |
| **Edge Functions** | Vercel Edge Functions | Cloudflare Workers | Lambda@Edge |
| **Next.js support** | ⭐ Native (made by Vercel) | Good | Manual config |
| **Price** | Generous free tier | Very generous free tier | Pay-as-you-go |
| **Global POPs** | ~30 | **300+** | 450+ |
| **Best For** | Next.js projects | High-traffic, global | Enterprise/complex AWS |

---

## HTTP/2 and HTTP/3

Modern CDNs support HTTP/2 and HTTP/3, enabling:
- **HTTP/2 Multiplexing**: Multiple requests over a single connection (eliminates head-of-line blocking).
- **HTTP/3 (QUIC)**: UDP-based transport, faster connection establishment, better packet loss recovery.

You get these automatically when deploying to Vercel or Cloudflare.

---

## Related Topics
- [[Build and deployment MOC]]
- [[CDN usage in frontend]]
- [[Network optimization patterns]]
- [[Browser and HTTP cache]]
- [[CI/CD for frontend]]
