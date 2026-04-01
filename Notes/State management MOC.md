---
created: 2026-04-01
tags: [moc]
related: [ [[Frontend system design MOC]], [[Local state in React]], [[Global state management]], [[Server state management]], [[Derived state and memoization]] ]
status: active
---

# State management MOC

## Overview

State management is the discipline of deciding **where data lives, how it changes, and how changes propagate** across your application's UI.

Choosing the wrong state management approach is one of the most common mistakes in React development. The most important insight:

> **Not all state is the same.** Different types of state have different lifetimes, owners, and update patterns. They require different tools.

---

## The Four Types of State

```
┌─────────────────────────────────────────────────────┐
│                   State Types                       │
├─────────────┬───────────────┬───────────────────────┤
│  Local      │    Global     │  Server     │ URL     │
│             │               │             │         │
│  useState   │  Zustand      │  TanStack   │ Search  │
│  useReducer │  Redux        │  Query      │ params  │
│  Form state │  Context API  │  SWR        │ Route   │
│             │  Jotai        │  Apollo     │         │
└─────────────┴───────────────┴─────────────┴─────────┘
```

### Local State
Data that only one component cares about — form field values, toggle states, modal open/close.

→ [[Local state in React]]

### Global (Client) State
Data shared across many components that has no server equivalent — UI theme, sidebar open/closed, selected tab.

→ [[Global state management]] — Zustand, Redux Toolkit, Context API

### Server State
Data that originates from the server and needs to be kept in sync — user profiles, product listings, API data.

→ [[Server state management]] — TanStack Query, SWR

### URL / Route State
Data stored in the URL — search queries, filters, pagination page. Always prefer URL state for shareable or bookmarkable UI states.

---

## The Most Common Mistake: Treating Server State as Client State

Before `TanStack Query` and `SWR`, developers put API data into Redux:

```typescript
// ❌ Anti-pattern: API data in Redux
dispatch(setOrders(await fetchOrders())) // Manual fetch → store → component
// You then have to manually handle: loading, error, caching, refetching, stale data
```

This is wrong because server state has unique properties that client state tools don't handle:
- It can be stale (out of sync with the server).
- It can be shared by multiple users simultaneously.
- It needs background refetching.
- It needs caching with invalidation logic.

TanStack Query handles all of this automatically.

---

## Decision Guide: Which Tool for Which State?

| Question | Answer | Use |
| :--- | :--- | :--- |
| Is it data from an API/server? | Yes | TanStack Query / SWR |
| Does only one component need it? | Yes | `useState` / `useReducer` |
| Is it shared, but not from server? | Yes | Zustand / Context |
| Is it a complex event-driven system? | Yes | Redux Toolkit |
| Should it be bookmarkable in URL? | Yes | URL search params |

---

## Sub-Topics (Deep Dives)
- [[Local state in React]] — `useState`, `useReducer`, complex local state
- [[Global state management]] — Zustand, Redux Toolkit, Context API, comparison
- [[Server state management]] — TanStack Query and SWR in depth
- [[Derived state and memoization]] — `useMemo`, selectors, avoiding unnecessary computation

---

## Related Notes
- [[Data fetching MOC]]
- [[React server components]]
- [[Frontend system design MOC]]
