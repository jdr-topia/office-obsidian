---
created: 2026-04-03
tags: [caching, service-worker, pwa, offline]
related: [ [[Caching strategies MOC]], [[Progressive Web Apps (PWA)]], [[Offline support MOC]] ]
status: active
---

# Service worker cache

A **Service Worker** is a JavaScript file that runs in the background, in a separate thread from the main browser window. It acts as a programmable network proxy — intercepting every request your page makes and deciding how to respond.

## What Can It Do?

- Intercept and cache network requests.
- Serve cached assets even when the user is **completely offline**.
- Run background sync and push notifications.
- Implement complex caching strategies impossible with HTTP headers alone.

---

## Lifecycle

```
Install → Activate → Fetch (intercept)
```

1. **Install**: The SW downloads and caches the "app shell" (HTML, CSS, JS).
2. **Activate**: The old SW is replaced. Stale caches are cleaned up.
3. **Fetch**: Every network request is intercepted. The SW decides: cache or network?

---

## Caching Strategies

### 1. Cache First (Offline First)
Check cache first. If not found, go to network. Great for static assets.
```js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      return cached || fetch(event.request);
    })
  );
});
```

### 2. Network First
Always try the network. Fall back to cache on failure. Great for API data.
```js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request).catch(() => caches.match(event.request))
  );
});
```

### 3. Stale While Revalidate
Serve from cache immediately, then update the cache in the background.
```js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('dynamic').then((cache) => {
      return cache.match(event.request).then((cached) => {
        const networkFetch = fetch(event.request).then((response) => {
          cache.put(event.request, response.clone());
          return response;
        });
        return cached || networkFetch;
      });
    })
  );
});
```

---

## Workbox: The Practical Approach

Writing service worker logic by hand is error-prone. **Workbox** (by Google) provides a high-level API for common caching strategies.

```js
import { registerRoute } from 'workbox-routing';
import { CacheFirst, StaleWhileRevalidate } from 'workbox-strategies';

// Cache images with Cache First
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({ cacheName: 'images' })
);

// Cache API calls with Stale While Revalidate
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new StaleWhileRevalidate({ cacheName: 'api-cache' })
);
```

---

## Next.js: `next-pwa`

The `next-pwa` package wraps Workbox for easy integration:

```js
// next.config.js
const withPWA = require('next-pwa')({ dest: 'public' });
module.exports = withPWA({ /* your next config */ });
```

---

## Related Topics
- [[Caching strategies MOC]]
- [[Offline support MOC]]
- [[Progressive Web Apps (PWA)]]
- [[Stale-while-revalidate]]
