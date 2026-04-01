---
created: 2026-04-01
tags: [moc]
related: [ [[Frontend system design MOC]], [[Monolith vs micro-frontend]], [[SPA vs MPA]], [[Backend for Frontend (BFF)]], [[CDN usage in frontend]], [[API gateway integration]] ]
status: active
---

# High level architecture MOC

## Overview

High Level Architecture defines the **structural blueprint** of your frontend system before writing any code. It answers the critical question: *How is this system organized and how do its parts communicate?*

A poor architecture decision made early is the most expensive type of technical debt.

---

## Architectural Dimensions

Every frontend system design involves decisions along these dimensions:

### 1. Team & Deployment Model
> How many teams own this system? Can parts deploy independently?
- [[Monolith vs micro-frontend]] — Single app vs. federated mini-apps

### 2. App Composition Model
> Is there one page or many? Does the browser or server do the routing?
- [[SPA vs MPA]] — Single Page App vs. Multi-Page App

### 3. Rendering Location
> Where does HTML get generated — browser, server, or build-time?
- [[Client side rendering (CSR)]]
- [[Server side rendering (SSR)]]
- [[Static site generation (SSG)]]
- → All covered in [[Rendering strategies MOC]]

### 4. Network Layer
> How does the frontend talk to the backend? Who orchestrates API calls?
- [[Backend for Frontend (BFF)]] — A dedicated server layer for frontend API needs
- [[API gateway integration]] — Routing and aggregation at the network layer

### 5. Content Delivery
> How is the frontend served globally with low latency?
- [[CDN usage in frontend]] — Distributing static assets globally

---

## Architecture Decision Matrix

| Question | Option A | Option B | Deciding Factor |
| :--- | :--- | :--- | :--- |
| One or many teams? | Monolith | Micro-frontend | Team count & autonomy |
| SPA or traditional web? | SPA | MPA | User journey type |
| Where to render? | CSR | SSR/SSG | SEO & performance needs |
| BFF or direct API calls? | Direct | BFF | API complexity |
| Self-hosted or CDN? | Origin server | CDN + Edge | User geography |

---

## Common Architecture Patterns

### Pattern 1: Standard Next.js Monolith
Best for small to medium teams (2-15 devs) and most products.

```
Browser
  ↕ HTTPS
Next.js App (Vercel / Node Server)
  → Server Components: Direct DB/API access
  → Client Components: Browser interactivity
  ↕ HTTP
Backend API (Node + Express + MongoDB)
```

### Pattern 2: BFF + Micro-frontend
Best for large teams with multiple specialized products.

```
Browser
  ↕ HTTPS
Shell App (Next.js, routing hub)
  ├── Auth App (independent React app)
  ├── Dashboard App (independent React app)
  └── Checkout App (independent React app)
           ↕ HTTP
BFF Layer (Node server, per team)
           ↕ gRPC / HTTP
Microservices (Auth, Orders, Products, etc.)
```

---

## Related Notes
- [[Monolith vs micro-frontend]]
- [[SPA vs MPA]]
- [[Backend for Frontend (BFF)]]
- [[CDN usage in frontend]]
- [[API gateway integration]]
- [[Rendering strategies MOC]]
