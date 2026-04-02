---
created: 2026-04-02
tags: [moc, routing]
related: [ [[Frontend system design MOC]] ]
status: active
---

# Routing strategy MOC

Routing is the backbone of navigation in modern web applications. It determines how URLs map to UI states and how users move between different views.

## Core Navigation Concepts
- [[Client side routing]]: Navigation without full page reloads (SPA model).
- **Server Side Routing**: Traditional multi-page application (MPA) navigation.
- [[Nested and dynamic routes]]: Building layouts that scale with content depth.
- [[Protected routes]]: Restricting access based on authentication or authorization.

## Performance & Optimization
- [[Route based code splitting]]: Loading logic only when the route is visited.
- **Prefetching**: Improving perceived performance by loading upcoming routes ahead of time.
- **Scroll Restoration**: Maintaining user position between navigations.

## Implementation Patterns
- **History API (BrowserRouter)** vs **Hash-based Routing (HashRouter)**
- **Path-to-RegExp**: How libraries (React Router, Next.js) parse URLs.
- **Query Parameters & State Management**: Using the URL as a source of truth for UI state.

---
## Related Topics
- [[Frontend system design MOC]]
- [[Authentication and authorization MOC]]
- [[Performance optimization MOC]]
