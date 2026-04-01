---
created: 2026-04-01
tags: [concept]
related: [ [[Requirement analysis in frontend design]], [[Functional vs non-functional requirements]], [[Monolith vs micro-frontend]], [[Frontend system design MOC]] ]
status: active
---

# Scalability in frontend systems

## Overview

Scalability in frontend systems is the ability of your application to handle **growth** — in users, data volume, team size, and feature complexity — without degrading in performance or maintainability.

Most engineers think of scalability as a backend problem (more servers, bigger database). But frontend scalability is just as critical, and it has two distinct dimensions:

1. **Runtime scalability**: Can the UI handle a million users and massive data sets?
2. **Development scalability**: Can a team of 50 developers work on this codebase without stepping on each other?

---

## Dimension 1: Runtime Scalability

This is about how the application performs under heavy load and large data volumes.

### The Core Problems

| Problem | Symptom | Example |
| :--- | :--- | :--- |
| Large JS bundle | 10s load time on 3G | Entire app downloaded upfront |
| Unmanaged re-renders | UI freezes on interaction | 5000-row table re-renders on every keystroke |
| No virtualization | Browser OOM crash | Rendering 100,000 items in a list |
| No caching | Every navigation fetches API again | Repeated network calls for static data |

### Solutions

**1. Code Splitting**
Only load the JavaScript needed for the current view.

```typescript
// Next.js handles this automatically per page with App Router.
// For manual splitting within a page, use dynamic imports:
import dynamic from 'next/dynamic'

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false, // Don't render on server if it uses browser APIs
})
```

**2. Virtualization**
Render only the items visible in the viewport, not all 100,000.

```typescript
// Using TanStack Virtual for a virtualized list
import { useVirtualizer } from '@tanstack/react-virtual'

function OrderList({ orders }: { orders: Order[] }) {
  const parentRef = React.useRef<HTMLDivElement>(null)

  const rowVirtualizer = useVirtualizer({
    count: orders.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 60, // estimated row height in px
  })

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: `${rowVirtualizer.getTotalSize()}px`, position: 'relative' }}>
        {rowVirtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {orders[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  )
}
```

---

## Dimension 2: Development Scalability

This is about how the codebase scales as the team grows.

### The Core Problems

| Problem | Symptom |
| :--- | :--- |
| No architectural boundaries | One team's change breaks another team's feature |
| Shared mutable global state | Impossible to track bugs across components |
| No component standards | Every developer writes UI differently |
| Monolithic deployments | Entire app must be redeployed for one feature change |

### Solutions

**1. Micro-frontend Architecture**
Split the app into independently deployable frontend applications.

> See: [[Monolith vs micro-frontend]] for a full comparison.

**2. Monorepo with Nx or Turborepo**
Manage multiple frontend packages in one repo with dependency awareness.

> See: [[Monorepo architecture]]

**3. Design Systems**
A shared component library that all teams use — enforces UI consistency.

> See: [[Design systems MOC]]

**4. Strict State Management Boundaries**
Each mini-app / feature owns its own state. Avoid a single global Redux store for the whole application.

---

## Scalability Planning by App Size

| App Size | Team | Recommended Architecture |
| :--- | :--- | :--- |
| Small (< 10k users, 2-5 devs) | Small | Monolith Next.js app |
| Medium (< 100k users, 5-15 devs) | Medium | Monolith with feature-sliced design |
| Large (> 1M users, 15+ devs) | Multiple teams | Micro-frontends, Monorepo |

---

## Key Principle: Don't Over-Engineer Early

A small startup does **not** need a micro-frontend architecture on day one. Scalability solutions add complexity and operational overhead. The rule is:

> **Build for your current scale, design for the next 2x.**

Only adopt micro-frontends or module federation when you genuinely have multiple independent teams that need autonomous deployment cycles.

---

## Related Notes
- [[Monolith vs micro-frontend]]
- [[Monorepo architecture]]
- [[Module federation]]
- [[Code splitting and lazy loading]]
- [[Virtualization]]
- [[Performance requirements]]
