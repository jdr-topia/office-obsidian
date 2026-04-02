---
created: 2026-04-02
tags: [concept]
related: [ [[Data fetching strategy MOC]], [[Server state management]], [[Backend for Frontend (BFF)]], [[Frontend system design MOC]] ]
status: active
---

# REST API vs GraphQL in frontend

## Overview

A fundamental decision in frontend system design is choosing how your UI will communicate with the backend services. **REST** and **GraphQL** are the two most prominent architectural styles for web APIs.

Each has its own philosophy, design patterns, and implications for performance, caching, and developer experience.

---

## 1. REST (Representational State Transfer)

An architectural style that treats resources (e.g., `/users`, `/products`, `/orders`) as first-class citizens. Uses standard HTTP methods (GET, POST, PUT, DELETE).

### Key Characteristics
- **Resource-Oriented**: Each URL represents a resource.
- **Stateless**: Each request must contain all the information necessary to understand it.
- **Cacheable**: Standard HTTP caching mechanisms apply naturally.

### Pros
- **Simplicity**: Easy to understand, standard conventions.
- **Caching**: Native HTTP caching (using `ETags`, `Cache-Control`).
- **Mature Ecosystem**: Every tool, library, and framework supports REST.
- **Client Independence**: Changes to one resource don't affect others.

### Cons
- **Over-fetching**: Returns all fields of a resource, even if the client only needs one.
- **Under-fetching**: One request often isn't enough (needs multiple calls to get related data).
- **Endpoint Explosion**: As UI diversity grows, backends often end up with dozens of custom endpoints (e.g., `/user-for-header`, `/user-for-profile`).

---

## 2. GraphQL

A query language for APIs and a runtime for fulfilling those queries with your existing data.

### Key Characteristics
- **Client-Driven**: The client defines the exact structure of the response.
- **Single Endpoint**: Typically uses one POST endpoint (e.g., `/graphql`).
- **Strongly Typed**: Based on a schema that defines all possible queries and mutations.

### Pros
- **No Over-fetching/Under-fetching**: Fetch only what you need in a single request.
- **Rapid UI Iteration**: Frontend can change data requirements without backend changes.
- **Self-Documenting**: Introspection allows tools like GraphiQL/Apollo Studio to provide live documentation.
- **Type Safety**: Unified schema between frontend and backend.

### Cons
- **Caching Complexity**: HTTP caching is harder because every request is a POST. Requires client-side caching (e.g., Apollo Client, TanStack Query).
- **Backend Complexity**: Resolvers can lead to performance issues (N+1 query problem) if not carefully implemented.
- **Learning Curve**: Requires understanding schemas, queries, mutations, and fragments.
- **File Uploads**: Not natively supported; requires workarounds (e.g., multipart requests).

---

## Comparative Matrix

| Feature | REST | GraphQL |
| :--- | :--- | :--- |
| **Data Fetching** | Fixed, server-side defined | Flexible, client-side defined |
| **Endpoint(s)** | Multiple URLs (resources) | Single URL (endpoint) |
| **Caching** | Native HTTP caching | Client-side/Normalized caching |
| **Typing** | Optional (via Swagger/OpenAPI) | Mandatory (schema-driven) |
| **Error Handling** | Standard HTTP Status Codes | `200 OK` with `errors` array |
| **Versioning** | Often `/v1/`, `/v2/` | Versionless (field deprecation) |

---

## When to use which?

### Use REST when:
- Your app is simple or small.
- You need deep, native HTTP caching.
- You have many third-party clients you don't control.
- Your backend is optimized for resource-based access.

### Use GraphQL when:
- You have complex, deeply nested data.
- You have multiple frontend clients (web, mobile, smartwatches) with vastly different data needs.
- You want a unified, type-safe API for a microservices architecture.
- You need high-speed UI iteration without constant backend updates.

---

## Hybrid Strategy: GraphQL Wrapper / BFF

A common modern architecture is to use a **GraphQL layer as a BFF (Backend for Frontend)** that aggregates data from multiple REST microservices.

```
Browser
   ↓ GraphQL Query
GraphQL Gateway (BFF)
   ├─ GET /users (REST)
   ├─ GET /orders (REST)
   └─ GET /products (REST)
   ↓ Aggregated Result
Browser
```

This gives you the client-side benefits of GraphQL while keeping the backend services simple and resource-oriented.
→ See: [[Backend for Frontend (BFF)]]

---

## Next.js Implementation Example

### REST Fetching (Server Component)

```typescript
// app/products/[id]/page.tsx
async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`)
  return res.json()
}

export default async function Page({ params }) {
  const product = await getProduct(params.id)
  return <div>{product.name}</div>
}
```

### GraphQL Fetching (with `gql`)

```typescript
// app/products/[id]/page.tsx
const GET_PRODUCT = `
  query GetProduct($id: ID!) {
    product(id: $id) {
      name
      price
      description
    }
  }
`

async function getProduct(id: string) {
  const res = await fetch('https://api.example.com/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      query: GET_PRODUCT,
      variables: { id },
    }),
  })
  const { data } = await res.json()
  return data.product
}

export default async function Page({ params }) {
  const product = await getProduct(params.id)
  return <div>{product.name}</div>
}
```

---

## Related Notes
- [[Data fetching strategy MOC]]
- [[Backend for Frontend (BFF)]]
- [[Server state management]]
- [[Frontend system design MOC]]
