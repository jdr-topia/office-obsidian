---
created: 2026-04-01
tags: [concept]
related: [ [[Rendering strategies MOC]], [[Hydration and partial hydration]], [[Static site generation (SSG)]], [[React server components]], [[Performance requirements]], [[Frontend system design MOC]] ]
status: active
---

# Islands architecture

## Overview

**Islands Architecture** is a rendering approach where the page is rendered as **mostly static HTML**, with isolated interactive components ("islands") embedded within it. Each island is an independently hydrated unit of interactivity. Everything outside an island is plain, non-interactive HTML.

The term was coined by **Katie Sylor-Miller** and popularized by **Astro.js**, though the concept applies broadly.

The core idea: **most web pages are 90% static content. Why ship and hydrate JavaScript for the 90% that never changes?**

---

## The Mental Model

```
Traditional SPA / SSR + Full Hydration:
  ┌─────────────────────────────────────────┐
  │ 🌊 OCEAN OF JAVASCRIPT (entire page)   │
  │   Everything hydrates. Buttons,         │
  │   paragraphs, images — all in the       │
  │   React tree. All JS is sent.           │
  └─────────────────────────────────────────┘

Islands Architecture:
  ┌─────────────────────────────────────────┐
  │  Static HTML (no JS cost)               │
  │  ┌───────────┐                          │
  │  │ 🏝 Island │  ← Interactive widget   │
  │  │ (JS sent) │     (search bar)         │
  │  └───────────┘                          │
  │  More static content...                 │
  │       ┌─────────────┐                   │
  │       │ 🏝 Island   │ ← Shopping cart  │
  │       │ (JS sent)   │                   │
  │       └─────────────┘                   │
  │  Footer content (no JS)                 │
  └─────────────────────────────────────────┘
```

---

## Why It Matters

On a typical content website (a blog, a documentation site, a marketing page):
- 80-90% of the content is **static**: headings, paragraphs, images, links.
- 10-20% is **interactive**: a search bar, a dark mode toggle, a newsletter form, a shopping cart.

With traditional React SSR, all 100% is sent as JavaScript and hydrated. With Islands, only the 10-20% interactive parts ship JS.

**Result**: Dramatically smaller JavaScript payloads and near-instant Time to Interactive (TTI).

---

## Islands in Practice: Astro

Astro is the framework most associated with Islands Architecture. It generates static HTML by default and lets you opt specific components into hydration:

```astro
---
// pages/blog/[slug].astro
import BlogContent from '../components/BlogContent.astro'  // Static — no JS
import SearchBar from '../components/SearchBar.jsx'         // Interactive island
import NewsletterForm from '../components/NewsletterForm.jsx'  // Interactive island
---

<html>
  <body>
    <nav>...</nav>                   <!-- Static HTML, zero JS -->

    <BlogContent />                  <!-- Static — rendered to HTML, no hydration -->

    <!-- These are "islands" — they ship and hydrate JS -->
    <SearchBar client:load />        <!-- Hydrate immediately on page load -->
    <NewsletterForm client:visible /> <!-- Hydrate only when scrolled into view -->
  </body>
</html>
```

**Astro's hydration directives** give fine-grained control:

| Directive | When hydrated |
| :--- | :--- |
| `client:load` | Immediately when page loads |
| `client:idle` | When the browser is idle (no critical tasks) |
| `client:visible` | When the component enters the viewport |
| `client:media="(max-width: 768px)"` | When a media query matches |
| `client:only` | Client-side only, no SSR at all |

---

## Islands Architecture in Next.js

Next.js doesn't use "Islands" terminology, but its **React Server Components + Client Components** model achieves the same concept:

```typescript
// app/blog/[slug]/page.tsx — The "static ocean"
// This entire page is predominantly Server Components (no JS hydration)

import BlogArticle from './BlogArticle'       // Server Component — static HTML
import TableOfContents from './TableOfContents' // Server Component — static HTML
import SearchWidget from './SearchWidget'       // 🏝 Island (Client Component)
import CommentForm from './CommentForm'         // 🏝 Island (Client Component)

export default async function BlogPostPage({ params }) {
  const post = await getPost(params.slug)

  return (
    <article>
      {/* Static HTML — zero JS payload on client */}
      <TableOfContents headings={post.headings} />
      <BlogArticle content={post.content} />

      {/* Islands — these ship JS and hydrate */}
      <SearchWidget />     {/* 'use client' component */}
      <CommentForm postId={post.id} />  {/* 'use client' component */}
    </article>
  )
}
```

```typescript
// app/blog/[slug]/SearchWidget.tsx — An "island"
'use client'
import { useState } from 'react'

export default function SearchWidget() {
  const [query, setQuery] = useState('')
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search posts..."
    />
  )
}
```

### Lazy Islands with `dynamic`

You can further optimize by deferring island hydration until the island is visible:

```typescript
import dynamic from 'next/dynamic'

// This island only loads its JS when the user scrolls to it
const CommentSection = dynamic(() => import('./CommentSection'), {
  loading: () => <p>Loading comments...</p>,
})

export default async function BlogPostPage({ params }) {
  const post = await getPost(params.slug)
  return (
    <article>
      <BlogContent content={post.content} /> {/* Static */}
      <CommentSection postId={post.id} />    {/* Lazy island */}
    </article>
  )
}
```

---

## Islands vs React Server Components: Key Difference

| Concept | Islands (Astro) | RSC (Next.js) |
| :--- | :--- | :--- |
| **Model** | Island-in-HTML | Component tree with Server/Client split |
| **Server rendering** | Full page as HTML | Full page as HTML |
| **Interactivity opt-in** | `client:load`, `client:visible` | `'use client'` directive |
| **Framework** | Astro (can use React, Vue, Svelte) | Next.js (React only) |
| **Best for** | Content-heavy sites, multi-framework | React-centric apps |

---

## When to Use Islands Architecture

| Scenario | Islands Architecture? | Why |
| :--- | :--- | :--- |
| Blog / docs / marketing site | ✅ Yes | Mostly static, very little interactivity |
| E-commerce product page | ✅ Yes | Static content + isolated interactive cart |
| Dashboard / admin panel | ❌ No | Highly interactive — everything is an island |
| Social feed app | ❌ Partially | Hybrid — consider Next.js RSC model |

---

## Related Notes
- [[Hydration and partial hydration]]
- [[Static site generation (SSG)]]
- [[React server components]]
- [[Performance requirements]]
- [[Rendering strategies MOC]]
