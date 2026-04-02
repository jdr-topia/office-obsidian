---
created: 2026-04-02
tags: [concept, performance]
related: [ [[Performance optimization MOC]], [[Performance requirements]], [[CDN usage in frontend]], [[Frontend system design MOC]] ]
status: active
---

# Network optimization patterns

## Overview

The network is the primary bottleneck for most web applications. Between high latency on mobile and limited bandwidth, **how and when** you request data and assets is critical.

Optimizing the network isn't just about compression. It's about **intelligent delivery, caching, and anticipating user needs**.

---

## 1. HTTP Caching

Reusing resources that have already been downloaded.

### Key Headers
| Header | Description | Status |
| :--- | :--- | :--- |
| **`Cache-Control`** | Specifies directives for caching mechanisms in both browsers and shared caches (e.g. Proxies, CDNs). | `public, max-age=31536000` |
| **`ETag`** | An identifier for a specific version of a resource. Browser sends `If-None-Match: ETag`. | `304 Not Modified` |
| **`Expires`** | An absolute date/time after which the response is considered stale. | (Optional, `Cache-Control` preferred) |

### Best Practice: Immutable Assets
For built assets (JS/CSS) with content-hashes in their names, set long-term caching:
- `Cache-Control: public, max-age=31536000, immutable`

---

## 2. Resource Hints (Rel)

Tell the browser about upcoming needs before they are formally required.

### `dns-prefetch`
Resolve the DNS of a domain in advance.
```html
<link rel="dns-prefetch" href="https://api.example.com" />
```

### `preconnect`
Establish a full connection (including TLS handshake) to a server in advance.
```html
<link rel="preconnect" href="https://api.example.com" crossorigin />
```

### `preload`
Tell the browser to download a high-priority resource ASAP.
```html
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
```

### `prefetch`
Download a low-priority resource that will likely be needed for the **next navigation**.
```html
<link rel="prefetch" href="/dashboard/stats" />
```
> **Next.js**: The `<Link>` component automatically prefetches linked pages when they enter the viewport.

---

## 3. Compression

Reducing the size of data before transmission.

- **Gzip**: Standard. Supported by all browsers and servers.
- **Brotli**: Modern. Up to 15-20% better compression than Gzip. (Next.js/Vercel use Brotli by default).
- **Format-specific**: Use WebP for images, WOFF2 for fonts.

---

## 4. HTTP/2 and HTTP/3

Modern protocols that solve the problems of older ones.

- **Multiplexing**: Send multiple requests over a single connection in parallel (no more "head-of-line blocking").
- **Server Push**: (H2 only, deprecated in many places) Proactively push assets.
- **QUIC (H3)**: Uses UDP instead of TCP. Eliminates handshake latency and handles packet loss/switching networks much better.

---

## 5. Early Hints (HTTP 103)

The server can send a "103 Early Hints" response with `<link rel="preload">` headers **immediately** upon receiving a request — even while it's still processing the final HTML response.

**Timeline**:
1. User requests `/product/1`.
2. Server sends "103 Early Hints" with preloads for CSS/Fonts. (10ms)
3. Browser starts downloading CSS/Fonts while server fetches product data.
4. Server sends final "200 OK" with full HTML. (500ms)
5. CSS/Fonts are already finished downloading. (500ms)

---

## Related Notes
- [[Performance optimization MOC]]
- [[Performance requirements]]
- [[CDN usage in frontend]]
- [[Asset optimization in Next.js]]
- [[Frontend system design MOC]]
