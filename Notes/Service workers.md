---
created: 2026-04-07
tags: [offline, service-worker, pwa, background-sync]
related: [ [[Offline support MOC]], [[Service worker cache]], [[Progressive Web Apps (PWA)]] ]
status: active
---

# Service workers

A **Service Worker** is a JavaScript script that the browser runs in a background thread, completely separate from the web page. It acts as a programmable proxy between the browser and the network, enabling offline support, asset caching, push notifications, and background sync.

> **Note**: For caching strategies (Cache First, Network First, Stale-While-Revalidate), see [[Service worker cache]]. This note focuses on the SW lifecycle and advanced features.

---

## Lifecycle in Depth

```
Register → Download → Install → Wait → Activate → Control
```

1. **Register**: The page calls `navigator.serviceWorker.register('/sw.js')`.
2. **Download**: The browser downloads `sw.js`.
3. **Install**: The `install` event fires. This is where you pre-cache the app shell.
4. **Wait**: If an old SW is active, the new one waits until all tabs are closed.
5. **Activate**: The `activate` event fires. Clean up old caches here.
6. **Control**: The SW now intercepts all `fetch` events for pages in its scope.

```js
// sw.js

const CACHE_NAME = 'app-shell-v1';
const APP_SHELL = ['/', '/index.html', '/main.css', '/main.js', '/offline.html'];

// Install: pre-cache the app shell
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(APP_SHELL))
  );
  self.skipWaiting(); // Activate immediately without waiting
});

// Activate: delete old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => k !== CACHE_NAME).map((k) => caches.delete(k)))
    )
  );
  self.clients.claim(); // Take control of all open tabs immediately
});
```

---

## Background Sync

Queues tasks (e.g., form submissions) when offline and replays them when connectivity is restored.

```js
// In the page — register a sync when submitting a form offline
async function submitForm(data) {
  try {
    await fetch('/api/submit', { method: 'POST', body: JSON.stringify(data) });
  } catch {
    // Offline — save to IndexedDB and register a background sync
    await saveToIndexedDB('pending-submissions', data);
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register('submit-form');
  }
}

// In the SW — handle the sync event when online
self.addEventListener('sync', (event) => {
  if (event.tag === 'submit-form') {
    event.waitUntil(replayPendingSubmissions());
  }
});
```

---

## Push Notifications

Service workers can receive push messages from the server even when the app is closed.

```js
// SW: listen for push events
self.addEventListener('push', (event) => {
  const data = event.data?.json();
  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: '/icon-192.png',
    })
  );
});
```

---

## Limitations

- **HTTPS only**: Service workers require a secure context (except `localhost`).
- **No DOM access**: Runs in a separate thread — cannot manipulate the DOM directly.
- **No synchronous storage**: Must use async APIs (Fetch, IndexedDB, Cache API).

---

## Related Topics
- [[Service worker cache]]
- [[Progressive Web Apps (PWA)]]
- [[Offline support MOC]]
