---
created: 2026-04-02
tags: [routing, spa]
related: [ [[Routing strategy MOC]], [[SPA vs MPA]] ]
status: active
---

# Client side routing

Client-side routing is the mechanism used by Single Page Applications (SPAs) to update the browser URL and the UI without a full page refresh.

## How it works

Instead of the browser asking the server for a new HTML document on every link click, the application intercepts the navigation event, updates the URL using the **History API**, and renders the appropriate component.

### The History API
- `history.pushState(state, title, url)`: Adds a new entry to the browser's session history stack.
- `history.replaceState(state, title, url)`: Modifies the current history entry.
- `popstate` event: Fired when the active history entry changes (e.g., user clicks the "Back" button).

## Pros & Cons

### Pros
- **Smooth transitions**: No white flashes between pages.
- **Shared State**: Global state (Redux/Context) is preserved across "navigations."
- **Reduced Server Load**: The server only serves the initial JS bundle and subsequent data (JSON) via APIs.

### Cons
- **Initial Load**: Larger JS bundle needed upfront (mitigate with [[Route based code splitting]]).
- **SEO Complexity**: Basic crawlers may see an empty `div#app` (mitigate with [[Server side rendering (SSR)]]).
- **Memory Management**: If not careful, long-running SPAs can accumulate memory leaks.

---

## Examples in Common Frameworks

### React (React Router)
```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
  </Routes>
</BrowserRouter>
```

### Next.js (App Router)
Next.js uses a hybrid model where initial page load is SSR/SSG, but subsequent navigations use client-side routing via the `<Link>` component.

---
## Related Topics
- [[SPA vs MPA]]
- [[Rendering strategies MOC]]
- [[Routing strategy MOC]]
