---
created: 2026-04-07
tags: [moc, offline, pwa, service-worker]
related: [ [[Frontend system design MOC]], [[Caching strategies MOC]] ]
status: active
---

# Offline support MOC

Offline support ensures your application remains functional (or at least informative) when the user loses network connectivity. This is a core requirement for Progressive Web Apps (PWAs) and any application used in unreliable network conditions.

## Layers of Offline Support

```
User goes offline
  └─ Service Worker intercepts requests
       ├─ Cached static assets (HTML, CSS, JS) → Served from cache
       ├─ Cached API responses → Served from cache (stale)
       └─ New uncached requests → Show offline fallback page
```

## Notes in this Section
- [[Service workers]]: The runtime engine that makes offline possible.
- [[Progressive Web Apps (PWA)]]: The full offline-capable app experience.

## The Spectrum of Offline Support

| Level | Description | Example |
|---|---|---|
| **Offline Shell** | App loads, shows cached UI shell | Any PWA |
| **Offline Data** | Previously viewed data is readable | News app, docs viewer |
| **Background Sync** | Queues actions taken offline and syncs when reconnected | Email drafts, form submissions |
| **Full Offline** | Complete functionality without any network | Google Docs offline mode |

---
## Related Topics
- [[Frontend system design MOC]]
- [[Caching strategies MOC]]
- [[Service worker cache]]
- [[Real time communication MOC]]
