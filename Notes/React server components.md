---
created: 2026-04-01
tags: [concept]
related: [ [[Frontend MOC]], [[Next.js Rendering Strategies (Complete Guide)]] ]
status: draft
---

# React server components

## Overview
React Server Components (RSC) are a new type of component that render on the server, separate from the client-side runtime. They allow for zero-bundle-size components and direct server-side data fetching.

## Key Principles
- **Server Side Rendering**: They only run on the server.
- **Zero Bundle Size**: No JavaScript for RSC is sent to the client.
- **Data Fetching**: Can fetch data directly from the database or filesystem.
- **Interoperability**: Can contain Client Components (marked with `"use client"`).

## Examples
```javascript
// A simple Server Component in Next.js
export default async function Page() {
  const data = await fetch('https://api.example.com/data');
  const items = await data.json();

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## References
- [[Next.js Rendering Strategies (Complete Guide)]]
- [[Frontend MOC]]
