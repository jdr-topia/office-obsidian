---
created: 2026-04-02
tags: [performance, routing, bundler]
related: [ [[Routing strategy MOC]], [[Performance optimization MOC]] ]
status: active
---

# Route based code splitting

Route-based code splitting is an optimization technique where a large application bundle is broken down into smaller chunks, with each chunk being loaded only when the corresponding route is visited.

## Why Do We Need It?

### The Problem: Monolithic Bundles
In a large SPA, if you bundle *everything* into a single `main.js` file, the user has to download the code for `/admin`, `/settings`, and `/profile` even if they only ever visit the homepage. This increases the **Initial Page Load Time** (TBT, LCP).

### The Solution: "Load on Demand"
Break the app into distinct bundles at the route level.

## How it works (React)

Using `React.lazy()` and `Suspense`.

```tsx
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// These components are now in separate .js files:
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading Page...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

---

## Performance Impacts

1.  **Reduced Bundle Size**: The browser only downloads the code for the active route.
2.  **Faster Interaction**: Decreases execution time during the initial boot.
3.  **Better Caching**: If the `/dashboard` code changes, the user doesn't have to re-download the code for `/home`.

## Optimization Tip: Prefetching

While splitting is great for size, it introduces a small delay when clicking a link (waiting for the chunk to download).
- **Solution**: Use `prefetch` (hover or intersection) to load the next route chunk in the background before the user actually clicks.

```tsx
// Next.js does this automatically for <Link> components in the viewport!
<Link href="/dashboard" prefetch={true}>Go to Dashboard</Link>
```

---
## Related Topics
- [[Code and bundle optimization]]
- [[Performance optimization MOC]]
- [[Routing strategy MOC]]
- [[Client side routing]]
- [[Hydration and partial hydration]]
- [[Image and font optimization]]
- [[HTTP caching and prefetching]]
