---
created: 2026-04-01
tags: [concept]
related: [ [[High level architecture MOC]], [[API gateway integration]], [[Data fetching MOC]], [[Server state management]], [[Frontend system design MOC]] ]
status: active
---

# Backend for Frontend (BFF)

## Overview

**BFF (Backend for Frontend)** is an architectural pattern where a **dedicated backend server is created for each frontend client** (web, mobile, etc.). Rather than the frontend calling multiple microservices directly, it calls one BFF which orchestrates all the required API calls and returns exactly the data the frontend needs.

The pattern was popularized by Sam Newman (author of "Building Microservices") and is used by Netflix, SoundCloud, and many others.

---

## The Problem BFF Solves

### Without BFF: The "chatty client" problem

In a microservices backend, a single user-facing page often requires data from multiple services:

```
Browser
  ↓ GET /user/profile     → User Service
  ↓ GET /user/orders      → Order Service
  ↓ GET /recommendations  → ML Recommendation Service
  ↓ GET /notifications    → Notification Service
```

**Problems with this approach**:
1. **Over-fetching**: Each service returns its full data model. Your Product card only needs `name`, `price`, and `imageUrl`, but the Product endpoint returns 50 fields.
2. **Under-fetching**: Sometimes one call isn't enough. You need the order, then the product details for each order item (N+1 problem).
3. **Network waterfall**: 4 sequential API calls. Each must wait for the previous.
4. **Coupling**: The frontend is tightly coupled to internal backend services. If microservice URLs or response shapes change, the frontend breaks.
5. **Authentication complexity**: Frontend must authenticate with every service individually.
6. **Mobile data concern**: Mobile devices on slow connections are fetching far more data than they display.

### With BFF: One optimized endpoint

```
Browser
  ↓ GET /api/dashboard    → BFF (Node server)
                                ↓ Calls User Service
                                ↓ Calls Order Service
                                ↓ Calls ML Service
                                ↓ Calls Notification Service
                                ↑ Aggregates and shapes the response
  ← Returns ONE perfect response for the Dashboard page
```

---

## Anatomy of a BFF

A BFF is typically a **Node.js server** (since frontend teams already know JavaScript) that:

1. Receives requests from the frontend.
2. Validates and authenticates the request.
3. Calls multiple downstream microservices (often in parallel).
4. Merges, transforms, and filters the responses.
5. Returns a response shaped exactly for the UI.

---

## Implementation: Building a Simple BFF in Node/Express

```typescript
// bff/src/routes/dashboard.ts
import express from 'express'
import { getUser } from '../services/userService'
import { getOrders } from '../services/orderService'
import { getRecommendations } from '../services/mlService'

const router = express.Router()

router.get('/dashboard', async (req, res) => {
  const userId = req.user.id // Extracted from auth middleware

  try {
    // Call all downstream services in parallel
    const [user, orders, recommendations] = await Promise.all([
      getUser(userId),
      getOrders(userId, { limit: 5 }),
      getRecommendations(userId, { limit: 10 }),
    ])

    // Shape the response exactly for the Dashboard UI
    res.json({
      profile: {
        name: user.fullName,
        avatar: user.avatarUrl,
      },
      recentOrders: orders.map((o) => ({
        id: o.orderId,
        status: o.currentStatus,
        total: o.grandTotal,
        date: o.createdAt,
      })),
      recommendations: recommendations.items.map((r) => ({
        productId: r.id,
        name: r.displayName,
        price: r.salePrice ?? r.basePrice,
        image: r.primaryImage,
      })),
    })
  } catch (error) {
    res.status(500).json({ message: 'Failed to load dashboard' })
  }
})

export default router
```

```typescript
// bff/src/services/userService.ts
// Internal service-to-service calls (not exposed to browser)
export async function getUser(userId: string) {
  const res = await fetch(`http://user-service/internal/users/${userId}`, {
    headers: { 'X-Internal-Token': process.env.INTERNAL_SERVICE_TOKEN },
  })
  return res.json()
}
```

---

## BFF with Next.js Route Handlers

In Next.js, **Route Handlers** (in `app/api`) naturally serve as a lightweight BFF layer:

```typescript
// app/api/dashboard/route.ts
import { NextResponse } from 'next/server'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'

export async function GET() {
  const session = await getServerSession(authOptions)
  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const userId = session.user.id

  // Parallel calls to backend services
  const [user, orders, recommendations] = await Promise.all([
    fetch(`${process.env.USER_SERVICE_URL}/users/${userId}`).then((r) => r.json()),
    fetch(`${process.env.ORDER_SERVICE_URL}/orders?userId=${userId}&limit=5`).then((r) => r.json()),
    fetch(`${process.env.ML_SERVICE_URL}/recommendations/${userId}`).then((r) => r.json()),
  ])

  return NextResponse.json({
    profile: { name: user.fullName, avatar: user.avatarUrl },
    recentOrders: orders.slice(0, 5).map((o) => ({
      id: o.orderId,
      status: o.currentStatus,
      total: o.grandTotal,
    })),
    recommendations: recommendations.items.slice(0, 10),
  })
}
```

```typescript
// In a Client Component or Server Component, you just call /api/dashboard
// Server Component (even simpler — direct fetch, no BFF needed in same Next.js app)
export default async function DashboardPage() {
  const data = await fetch('/api/dashboard').then((r) => r.json())

  return <Dashboard data={data} />
}
```

---

## Pros and Cons

### Pros
- Decouples frontend from backend microservice structure.
- Reduces client-side network calls (one optimized endpoint per view).
- Enables server-side data aggregation and response shaping.
- Frontend team owns and controls the BFF (faster iteration).
- Security: Internal microservice URLs never exposed to the browser.

### Cons
- Adds another service to maintain and deploy.
- Can become a "dumping ground" — teams add logic without discipline.
- Potential for BFF to become a monolith over time.
- Adds latency (extra hop from browser → BFF → services).

---

## BFF vs API Gateway

| Dimension | BFF | API Gateway |
| :--- | :--- | :--- |
| **Purpose** | Shape data for specific client | Route, throttle, auth at network level |
| **Owned by** | Frontend team | Platform/infra team |
| **Logic** | Data aggregation, transformation | Routing, rate limiting, auth |
| **One per** | Frontend client (web, mobile) | Entire platform |
| **Example** | `/api/dashboard` aggregating 4 services | Kong, AWS API Gateway, Nginx |

They often **complement each other**: API Gateway handles cross-cutting concerns (rate limiting, auth tokens), BFF handles domain-specific data shaping.

---

## Related Notes
- [[API gateway integration]]
- [[High level architecture MOC]]
- [[Data fetching MOC]]
- [[Server state management]]
- [[REST API integration]]
