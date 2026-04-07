---
created: 2026-04-07
tags: [offline, pwa, mobile, installable]
related: [ [[Offline support MOC]], [[Service workers]], [[Service worker cache]] ]
status: active
---

# Progressive Web Apps (PWA)

A **Progressive Web App** is a web application that uses modern web APIs to deliver an app-like experience: installable on desktop and mobile, works offline, receives push notifications, and loads instantly on repeat visits.

## The Three Pillars of a PWA

| Pillar | Technology |
|---|---|
| **Capable** | Web APIs (Camera, Geolocation, Bluetooth, File System) |
| **Reliable** | Service Workers + Cache API (offline/fast loads) |
| **Installable** | Web App Manifest |

---

## 1. The Web App Manifest

A JSON file that tells the browser how to present your app when installed.

```json
// public/manifest.json
{
  "name": "My Awesome App",
  "short_name": "AwesomeApp",
  "description": "The best app in the world.",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4A90E2",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

```html
<!-- Link it in your HTML head -->
<link rel="manifest" href="/manifest.json" />
<meta name="theme-color" content="#4A90E2" />
```

### `display` Options
| Value | Behavior |
|---|---|
| `standalone` | Looks like a native app, no browser chrome |
| `fullscreen` | Takes the entire screen |
| `minimal-ui` | Small browser controls shown |
| `browser` | Regular browser tab (not really a PWA) |

---

## 2. PWA in Next.js (`next-pwa`)

```bash
npm install next-pwa
```

```js
// next.config.js
const withPWA = require('next-pwa')({
  dest: 'public',           // SW is generated in /public
  disable: process.env.NODE_ENV === 'development', // Disable in dev
  register: true,           // Auto-register the SW
  skipWaiting: true,        // Activate new SW immediately
});

module.exports = withPWA({
  // ... other Next.js config
});
```

This automatically generates `sw.js` and `workbox-*.js` in your `public` folder on build.

---

## 3. Installability Criteria (Lighthouse Check)

For the browser to show the "Add to Home Screen" prompt, your app must:
- [ ] Be served over **HTTPS**.
- [ ] Have a valid **Web App Manifest** with name, icons, and `start_url`.
- [ ] Have a **registered Service Worker** with a `fetch` handler.

---

## PWA Audit Checklist

- [ ] Loads offline or on flaky networks.
- [ ] Provides a custom offline page.
- [ ] Is installable (passes Lighthouse PWA audit).
- [ ] Uses push notifications responsibly (ask permission contextually).
- [ ] Icons include a `maskable` variant for Android adaptive icons.
- [ ] `start_url` is cached and works offline.

---

## Related Topics
- [[Offline support MOC]]
- [[Service workers]]
- [[Service worker cache]]
- [[Performance optimization MOC]]
