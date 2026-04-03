---
created: 2026-04-03
tags: [caching, performance, ux, tanstack-query]
related: [ [[Caching strategies MOC]], [[Server state management]], [[Browser and HTTP cache]] ]
status: active
---

# Stale-while-revalidate

**Stale-while-revalidate (SWR)** is a caching strategy where the client immediately serves a cached (potentially stale) response to the user for instant display, while simultaneously fetching a fresh version of that data in the background to update the cache.

## The Core Principle

> "Show something immediately. Update it silently."

This eliminates the loading spinner for repeat visits — the user always sees content right away.

---

## The Problem It Solves

**Without SWR:**
1. User visits `/dashboard`.
2. Browser shows a spinner for 800ms while the API responds.
3. Content appears.
4. User navigates away and comes back.
5. Spinner again for 800ms.

**With SWR:**
1. User visits `/dashboard`.
2. Cached data is shown in **0ms** (from a previous visit).
3. A fresh fetch happens in the background.
4. When the fresh data arrives, the UI updates seamlessly (if different).

---

## As an HTTP Header

Defined in `Cache-Control`:

```http
Cache-Control: max-age=1, stale-while-revalidate=59
```

- For the first 1 second: the response is fresh, serve from cache.
- For seconds 2–60: response is stale but usable. Serve from cache AND revalidate in background.
- After 60 seconds: fully stale. Must revalidate before serving.

---

## In TanStack Query

TanStack Query implements SWR natively through `staleTime` and `gcTime`.

```ts
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 1000 * 60 * 5,  // Data is "fresh" for 5 minutes → no background refetch
  gcTime: 1000 * 60 * 10,    // Cached data is kept in memory for 10 minutes
});
```

**Default behaviour** (when `staleTime: 0`): Every time a component mounts, it serves the cached data instantly and immediately starts a background refetch.

---

## Refetch Triggers in TanStack Query

By default, a revalidation is triggered when:
- The component mounts (`refetchOnMount`)
- The window regains focus (`refetchOnWindowFocus`)
- The network reconnects (`refetchOnReconnect`)

```ts
useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
  refetchOnWindowFocus: false, // Disable if updates are infrequent
});
```

---

## Trade-offs

| Consideration | Impact |
|---|---|
| **Consistency** | Users may briefly see stale data before it updates |
| **Extra Requests** | More API calls than a pure cache-first approach |
| **UX** | Feels dramatically faster on repeat visits |
| **Best For** | Data that changes occasionally but must eventually be fresh |

---

## Related Topics
- [[Caching strategies MOC]]
- [[Server state management]]
- [[Browser and HTTP cache]]
- [[Service worker cache]]
- [[Incremental static regeneration (ISR)]]
