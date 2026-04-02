---
created: 2026-04-02
tags: [concept, user-experience]
related: [ [[Data fetching strategy MOC]], [[Performance requirements]], [[Rendering strategies MOC]], [[Frontend system design MOC]] ]
status: active
---

# Pagination and infinite scrolling

## Overview

When your frontend application needs to display a large dataset (e.g., thousands of products, millions of posts), you must decide how to fetch and present that data without overwhelming the browser or the network.

**Pagination** and **Infinite Scrolling** are the two primary patterns for handling large lists of data efficiently.

---

## 1. Traditional Pagination

The user navigates through data in discrete chunks (pages).

### Key Characteristics
- **Page Numbers**: Navigation via `1, 2, 3 ... Next`.
- **Fixed Size**: Each page contains a specific number of items.
- **Predictable**: User knows exactly where stay and can easily find a specific page.

### Pros
- **User Control**: Easy to jump to a specific section.
- **Bookmarkable**: Page 5 is always `?page=5`.
- **Sense of Progress**: Users can see how much content there is.
- **Better for Research/Search**: Finding a specific result is easier.

### Cons
- **Friction**: Requires a click/wait for every new page.
- **Disruptive**: Navigation and full/partial page re-renders can be slow.

### Implementation Patterns

**A. Offset-Based Pagination (Limit/Offset)**
Most common in SQL databases.
```sql
SELECT * FROM products LIMIT 20 OFFSET 40;
```
**Problem**: Slows down as pages get deeper (database must skip 40 rows). High risk of duplicates if data changes between page switches.

**B. Cursor-Based Pagination**
Uses a unique pointer (cursor) from the previous result.
```sql
SELECT * FROM products WHERE id > 100 LIMIT 20;
```
**Benefit**: Constant time performance, no duplicates. Preferred for real-time apps.

---

## 2. Infinite Scrolling

New data is automatically loaded as the user scrolls to the bottom of the current list.

### Key Characteristics
- **No Clicks**: Continuous discovery experience.
- **Bottomless**: Feels like an endless stream of content.
- **Engagement**: Keeps users on the page longer.

### Pros
- **High Retention**: People scroll more than they click.
- **Modern Experience**: Standard for social feeds (TikTok, Instagram).
- **Reduced Friction**: Seamless transition between data chunks.

### Cons
- **Hard to Find Footer**: User can't click links in the footer if it's always pushed down.
- **Back Button Pain**: Navigating back to the exact scroll position is famously hard in SPAs.
- **Memory Pressure**: Browser keeps thousands of DOM elements in memory (Solved via **Virtualization**).
- **Harder to Bookmark**: How do you share "halfway down the scroll"?

---

## 3. "Load More" Button (The Hybrid)

The user clicks a button to load more results.

### Pros
- **Friction-less**: Easier than pagination but more intentional than infinite scroll.
- **Footer Access**: Users can reach the footer.
- **Network Control**: User decides when to spend bandwidth on more data.

---

## Comparative Matrix

| Feature | Pagination | Infinite Scroll | Load More |
| :--- | :--- | :--- | :--- |
| **Best For** | Search, Admin, Data tables | Social, Discovery, Media | E-commerce, Portfolios |
| **User Intent** | Goal-oriented | Exploration-oriented | Balanced |
| **SEO** | Easy (individual pages) | Hard (requires JS) | Medium |
| **UX Friction** | High | Lowest | Low |

---

## Technical Implementation (Next.js & TanStack Query)

### Infinite Scroll Implementation

```typescript
// app/feed/page.tsx
'use client'
import { useInfiniteQuery } from '@tanstack/react-query'
import { useInView } from 'react-intersection-observer'
import { useEffect } from 'react'

async function fetchPosts({ pageParam = 1 }) {
  const res = await fetch(`/api/posts?page=${pageParam}`)
  return res.json()
}

export default function InfiniteScrollFeed() {
  const { ref, inView } = useInView()

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    status
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    getNextPageParam: (lastPage, allPages) => {
      // Logic to determine the next page number/cursor
      return lastPage.nextPage ?? undefined
    },
    initialPageParam: 1,
  })

  // Trigger fetchNextPage when the user scrolls to the bottom observer
  useEffect(() => {
    if (inView && hasNextPage) {
      fetchNextPage()
    }
  }, [inView, hasNextPage, fetchNextPage])

  if (status === 'pending') return <p>Loading...</p>

  return (
    <div>
      {data.pages.map((group, i) => (
        <React.Fragment key={i}>
          {group.posts.map((post) => (
            <div key={post.id} className="post-card">
              {post.title}
            </div>
          ))}
        </React.Fragment>
      ))}

      {/* The invisible observer element at the bottom */}
      <div ref={ref}>
        {isFetchingNextPage ? 'Loading more...' : hasNextPage ? 'Scroll for more' : 'End of content'}
      </div>
    </div>
  )
}
```

---

## Handling the Back Button Problem

If a user scrolls down 10 pages in an infinite list, clicks a post, and then clicks "Back", they expect to return to the 10th page's scroll position.

**Strategies**:
1. **Cache Persistence**: Use TanStack Query to keep the `posts` cache alive.
2. **Scroll Restoration**: Use a custom hook to save the `scrollTop` to `sessionStorage` and restore it on mount.
3. **Modal Pattern**: Open posts in a modal instead of a full page navigation, keeping the scroll position in the background.

---

## Related Notes
- [[Performance optimization MOC]]
- [[Virtualization]]
- [[Data fetching strategy MOC]]
- [[Server state management]]
- [[Frontend system design MOC]]
