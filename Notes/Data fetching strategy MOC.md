---
created: 2026-04-02
tags: [moc]
related: [ [[Frontend system design MOC]], [[Server state management]], [[REST API vs GraphQL in frontend]] ]
status: active
---

# Data fetching strategy MOC

## Overview

Data fetching is the bridge between your frontend UI and the backend services. A well-designed fetching strategy ensures that your application is **performant, resilient, and provides a smooth user experience**.

It's not just about calling `fetch()`. It involves handling loading states, errors, caching, retries, and background synchronization.

---

## Communication Protocols

### 1. REST (Representational State Transfer)
The most common architectural style for networked applications. Uses standard HTTP methods (GET, POST, PUT, DELETE).
→ [[REST API vs GraphQL in frontend]]

### 2. GraphQL
A query language for APIs that allows clients to request exactly the data they need.
→ [[REST API vs GraphQL in frontend]]

### 3. gRPC-web
A high-performance, open-source universal RPC framework. Uses Protocol Buffers as its interface definition language.

---

## Key Fetching Patterns

### 1. Caching & Deduplication
Reducing repetitive network calls by storing previously fetched data in a local cache.
→ See: [[Server state management]] (TanStack Query)

### 2. Optimistic Updates
Updating the UI immediately as if the request succeeded, before receiving a response from the server.
→ [[Optimistic updates and retry strategies]]

### 3. Resilience Strategies
- **Retries**: Automatically re-attempting failed requests (e.g., exponential backoff).
- **Timeouts**: Preventing requests from hanging indefinitely.
- **Circuit Breakers**: Stopping requests to a failing service to prevent cascading failures.
→ [[Optimistic updates and retry strategies]]

### 4. Efficient Loading
- **Pagination**: Loading data in small chunks (pages).
- **Infinite Scrolling**: Automatically loading more data as the user scrolls.
→ [[Pagination and infinite scrolling]]

---

## Data Fetching Tools

| Tool | Focus | Use When... |
| :--- | :--- | :--- |
| **Fetch API** | Low-level HTTP | Small apps, no dependencies |
| **Axios** | HTTP Client | Need interceptors, automatic JSON transform |
| **TanStack Query** | Server State Management | Need caching, retries, sync (Recommended) |
| **SWR** | Lightweight Caching | Simple caching needs |
| **Apollo Client** | GraphQL State Management | Complex GraphQL apps |

---

## Decision Guide: When to fetch?

### 1. Fetch on Render (CSR)
Data fetching happens inside a component (e.g., `useEffect`). Leads to waterfalls.
→ See: [[Client side rendering (CSR)]]

### 2. Fetch on the Server (SSR)
Data is fetched on the server before the HTML is sent to the client. Best for SEO and initial load.
→ See: [[Server side rendering (SSR)]]

### 3. Fetch in Parallel
Fetch multiple data sources simultaneously to avoid sequential "waterfalls."
→ See: [[Streaming SSR]]

---

## Related Notes
- [[Server side rendering (SSR)]]
- [[Client side rendering (CSR)]]
- [[Streaming SSR]]
- [[Server state management]]
- [[Frontend system design MOC]]
