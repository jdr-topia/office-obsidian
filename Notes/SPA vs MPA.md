---
created: 2026-04-01
tags: [concept]
related: [ [[High level architecture MOC]], [[Client side rendering (CSR)]], [[Server side rendering (SSR)]], [[Rendering strategies MOC]], [[Frontend system design MOC]] ]
status: active
---

# SPA vs MPA

## Overview

**SPA (Single Page Application)** and **MPA (Multi-Page Application)** describe how your application handles navigation and page transitions at a fundamental level.

This decision has cascading effects on rendering strategy, SEO, perceived performance, and architecture.

---

## Multi-Page Application (MPA)

The traditional web model. Each navigation request fetches a **completely new HTML document** from the server.

### How it works

```
User clicks "Products" link
         ↓
Browser sends GET /products to server
         ↓
Server renders the full HTML for /products page
         ↓
Browser discards current page, renders new HTML
         ↓
User sees the new page (full page reload)
```

### Examples
- Traditional PHP/Rails websites.
- Next.js pages when you don't use the client-side router (direct browser navigation).
- Any website with full server-side rendering per page.

### Pros
- **SEO-friendly by default**: Full HTML on every page, instantly crawlable.
- **Simpler architecture**: No client-side router, no complex state persistence.
- **Better initial load**: Each page only loads what it needs.
- **Natural code splitting**: Browser only downloads JS for the current page.

### Cons
- **Full page flicker**: Every navigation causes a flash of white as the browser discards the current page.
- **Repeated resource loading**: Header, fonts, and common scripts re-download on every page.
- **Slower perceived navigation**: Even with a fast server, the browser must do a full teardown and rebuild.
- **State loss**: Any in-memory state (e.g., form input, open modals) is lost on navigation.

---

## Single Page Application (SPA)

The modern web model. The browser downloads the app **once** and JavaScript handles all subsequent navigation by updating the DOM — never doing a full page reload.

### How it works

```
Initial Load:
  Browser loads index.html + JS bundle
          ↓
  React mounts and renders the current route

User clicks "Products" link:
  JavaScript intercepts the click
          ↓
  React Router changes the URL (pushState)
          ↓
  React renders the Products component
          ↓
  User sees the transition instantly (no reload)
  JavaScript fetches /api/products in the background
```

### Examples
- Pure React apps (`create-react-app`, Vite).
- Gmail, Google Maps, Notion, Figma.
- Next.js with client-side navigation via `<Link>`.

### Pros
- **Instant navigation feel**: Page transitions are immediate — app feels native.
- **Persistent state**: Auth state, open menus, notifications persist across navigation.
- **Rich interactions**: Complex animations, real-time updates, optimistic UI.
- **Reduced server load**: Server only responds to API calls, not full page renders.

### Cons
- **SEO challenges**: Initial HTML is empty (if pure CSR). Requires SSR/SSG for SEO.
- **Large initial bundle**: The entire app's JS is often downloaded on first visit.
- **Slow initial load**: Time to interactive can be high before the JS hydrates.
- **Complex state management**: You must manage routing, caching, and history yourself.
- **Accessibility challenges**: Focus management and browser back-button behaviour must be handled manually.

---

## The Hybrid (The Modern Approach): Next.js

In practice, most modern apps are hybrids that get the best of both worlds.

**Next.js** implements this hybrid model:
- Each page is a **full MPA request** on the first visit (full HTML from server = SSR/SSG).
- Subsequent navigations are **SPA-style** (client-side routing via `<Link>`) — no full reload.

```typescript
// This is the Next.js <Link> component
// First load: server renders full HTML (MPA-like)
// Subsequent navigation: client-side, instant (SPA-like)
import Link from 'next/link'

export default function Nav() {
  return (
    <nav>
      <Link href="/dashboard">Dashboard</Link>
      <Link href="/products">Products</Link>
    </nav>
  )
}
```

### Prefetching

Next.js makes this even faster by prefetching link targets in the background:

```typescript
// By default, Next.js prefetches all <Link> hrefs visible in the viewport
// This means clicking a link is nearly instant
// You can disable it for less critical links:
<Link href="/archive" prefetch={false}>Archive</Link>
```

---

## Comparison Table

| Dimension | MPA | SPA | Hybrid (Next.js) |
| :--- | :--- | :--- | :--- |
| **SEO** | Excellent | Poor (CSR) | Excellent |
| **Initial load** | Good (per page) | Slow (full bundle) | Excellent (SSR/SSG) |
| **Navigation speed** | Slow (full reload) | Instant | Instant |
| **State persistence** | None | Easy | Easy |
| **Architecture complexity** | Low | Medium | Medium |
| **Best for** | Content sites | Web apps | Everything else |

---

## When to Use Which

### Use MPA when:
- Simple content site with minimal interactivity (blogs, marketing pages).
- SEO is critical and the team doesn't want to manage hydration complexity.
- Using a server-side language/template engine where CSR adds no value.

### Use SPA when:
- Dashboard, tool, or editor with high client-side interactivity.
- App is behind a login (SEO irrelevant).
- Rich state that must persist across navigation (e.g., a multi-step form).

### Use Hybrid (Next.js) when:
- Almost everything else. This is the modern default for most production apps.

---

## Related Notes
- [[Client side rendering (CSR)]]
- [[Server side rendering (SSR)]]
- [[Static site generation (SSG)]]
- [[Rendering strategies MOC]]
- [[High level architecture MOC]]
