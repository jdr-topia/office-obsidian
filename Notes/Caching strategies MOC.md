---
created: 2026-04-03
tags: [moc, caching, performance]
related: [ [[Frontend system design MOC]], [[Performance optimization MOC]] ]
status: active
---

# Caching strategies MOC

Caching is the practice of storing copies of data so future requests can be served faster. In frontend systems, caching happens at multiple layers simultaneously.

## Caching Layers

| Layer | Where | Tools |
|---|---|---|
| **Browser/HTTP Cache** | Browser memory & disk | `Cache-Control`, ETags |
| **Service Worker Cache** | Browser background thread | Cache Storage API |
| **In-Memory / Client Cache** | JavaScript runtime | TanStack Query, SWR |
| **CDN Cache** | Edge servers globally | Vercel, Cloudflare |

## Notes in this Section
- [[Browser and HTTP cache]]: HTTP headers that control caching behavior.
- [[Service worker cache]]: Offline-capable caching via Service Workers.
- [[Stale-while-revalidate]]: Serving stale data instantly while refreshing in the background.

## Key Mental Model

> "Cache as close to the user as possible, invalidate as precisely as possible."

The hardest problem in caching is **invalidation** — knowing when to throw away stale data and fetch fresh data.

---
## Related Topics
- [[Frontend system design MOC]]
- [[Performance optimization MOC]]
- [[Network optimization patterns]]
- [[HTTP caching and prefetching]]
- [[Real time communication MOC]]
