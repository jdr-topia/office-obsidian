---
created: 2026-04-03
tags: [caching, http, performance, network]
related: [ [[Caching strategies MOC]], [[Network optimization patterns]], [[HTTP caching and prefetching]] ]
status: active
---

# Browser and HTTP cache

The browser has a built-in caching layer controlled entirely through **HTTP response headers**. When configured correctly, it is the cheapest and most powerful caching mechanism available — zero client-side code required.

## How It Works

When a browser makes a request and receives a response, it inspects the `Cache-Control` header to decide:
1. **Should I cache this response?**
2. **For how long?**
3. **Can I serve this from cache without asking the server?** (fresh vs stale)

---

## Key HTTP Headers

### `Cache-Control`
The primary directive for caching behavior.

| Directive | Meaning |
|---|---|
| `max-age=3600` | Cache is fresh for 3600 seconds (1 hour) |
| `no-cache` | Cache but always revalidate with server before using |
| `no-store` | Never cache (use for sensitive data like banking) |
| `public` | Any cache (CDN, proxy) can store this |
| `private` | Only the user's browser can store this |
| `immutable` | The file will never change; skip revalidation |
| `stale-while-revalidate=60` | Serve stale for up to 60s while refreshing in background |

```http
Cache-Control: public, max-age=31536000, immutable
```
☝️ This is the ideal header for **hashed static assets** (e.g., `app.a3f9b2.js`).

---

### `ETag` (Entity Tag)
A fingerprint (hash) of the file's content. Used for **conditional requests**.

```
Flow:
1. Server sends:  ETag: "abc123"
2. Browser caches the file and the ETag
3. Cache expires. Browser asks: GET /file, If-None-Match: "abc123"
4. Server checks: has content changed?
   - No  → 304 Not Modified (no body, saves bandwidth)
   - Yes → 200 OK with new content + new ETag
```

### `Last-Modified`
Similar to ETag but uses a timestamp. Less reliable than ETags (time zone issues, 1-second granularity).

---

## Practical Cache Strategy by Resource Type

| Resource | Strategy | Header |
|---|---|---|
| HTML pages | No cache / short `max-age` | `no-cache` |
| Hashed JS/CSS bundles | Cache forever | `max-age=31536000, immutable` |
| Images (unhashed) | Medium duration + ETag | `max-age=86400` + `ETag` |
| API responses | Short duration or no cache | `max-age=60` or `no-store` |
| Fonts | Very long duration | `max-age=31536000, immutable` |

---

## Next.js Caching

Next.js sets smart `Cache-Control` headers automatically:
- Static assets in `/_next/static/` get `max-age=31536000, immutable`.
- `fetch()` requests in Server Components are cached and deduplicated by default.

```ts
// Override fetch cache behavior in Next.js Server Components
fetch('/api/data', { cache: 'no-store' });          // Always fresh
fetch('/api/data', { next: { revalidate: 60 } });   // ISR-style: revalidate every 60s
```

---

## Related Topics
- [[Caching strategies MOC]]
- [[HTTP caching and prefetching]]
- [[Network optimization patterns]]
- [[Stale-while-revalidate]]
- [[Static site generation (SSG)]]
