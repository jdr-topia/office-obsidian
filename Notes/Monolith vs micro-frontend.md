---
created: 2026-04-01
tags: [concept]
related: [ [[High level architecture MOC]], [[Scalability in frontend systems]], [[Monorepo architecture]], [[Module federation]], [[Frontend system design MOC]] ]
status: active
---

# Monolith vs micro-frontend

## Overview

This is one of the most important architecture decisions in frontend system design. It determines how your codebase is **organized, deployed, and owned**.

---

## The Monolith

A **frontend monolith** is a single, unified application that owns all UI features. One codebase, one deployment, one team (or multiple teams on the same codebase).

### What it looks like

```
my-next-app/
├── app/
│   ├── dashboard/
│   ├── auth/
│   ├── orders/
│   └── settings/
├── components/
└── package.json          ← Single package
```

### Pros

- **Simple mental model**: One codebase to understand, one deployment pipeline.
- **Easy refactoring**: Rename a type or utility and all usages update at once.
- **Shared state is trivial**: No cross-application communication problems.
- **Low operational overhead**: One build process, one CI job.
- **Best for small teams**: 1-10 developers working closely together.

### Cons

- **Scaling bottleneck**: As the team grows, merging conflicts become frequent.
- **All-or-nothing deployments**: A bug in the checkout feature requires redeploying the entire app, including the auth feature.
- **Long build times**: The entire app is compiled even for a small change.
- **No team autonomy**: Multiple teams must coordinate on shared dependencies.

---

## Micro-frontend

A **micro-frontend** architecture splits the UI into multiple independently built, tested, and deployed applications. Each mini-app is owned by a separate team and represents a distinct business domain.

### What it looks like

```
# Each "app" is a separate repository/package:

shell-app/            ← Routing hub, loads all other apps
  └── package.json

auth-app/             ← Owned by the Auth team
  └── package.json

dashboard-app/        ← Owned by the Analytics team
  └── package.json

checkout-app/         ← Owned by the Commerce team
  └── package.json
```

At runtime, the **shell app** (also called the "host") loads the separate mini-apps, usually via **Module Federation** or iframes.

### Pros

- **Independent deployment**: The Checkout team deploys without waiting for the Auth team.
- **Team autonomy**: Each team can choose their own dependency versions (to a degree).
- **Isolated failures**: A crash in the `checkout-app` does not crash the `dashboard-app`.
- **Parallel development**: Multiple teams ship features simultaneously.
- **Technology flexibility**: One team can use Next.js while another uses SvelteKit (controversial but possible).

### Cons

- **Massive operational complexity**: Multiple CI/CD pipelines, versioning, cross-app communication protocols.
- **Shared state is hard**: Authentication state, user preferences — sharing data between apps requires a protocol.
- **Duplicated dependencies**: If both apps use React, both bundles include React (can be solved with shared libs in Module Federation, but it's complex).
- **Performance implications**: Loading multiple entry points instead of one coherent bundle.
- **Steep learning curve**: Module Federation, contract testing, and shared design systems add significant complexity.

---

## The Comparison

| Dimension | Monolith | Micro-frontend |
| :--- | :--- | :--- |
| **Team size** | 1-15 devs | 15+ devs, multiple teams |
| **Deployment** | One pipeline | Independent per mini-app |
| **Shared state** | Easy | Complex (event bus, shared store) |
| **Operational cost** | Low | High |
| **Build time** | Gets slow at scale | Fast per mini-app |
| **Tech stack** | Unified | Can differ per team |
| **Failure isolation** | Low | High |
| **Best for** | Startups, single products | Enterprise, platform products |

---

## When to Use Which

### Use a Monolith when:
- Your team is smaller than 10-12 developers.
- All features are closely coupled (share lots of state/components).
- You have one product, not a platform of products.
- You value speed of development over deployment independence.
- **Example**: A SaaS app for a startup, an internal admin tool.

### Use Micro-frontends when:
- You have 3+ independent teams that need to ship without coordinating with each other.
- Your product is a platform with distinct domains (e.g., `analytics`, `billing`, `marketplace`).
- You need isolated failure and independent rollbacks.
- **Example**: A large e-commerce platform, a banking portal with separate product teams.

> **Anti-pattern**: Do NOT start with micro-frontends. Even large companies (Netflix, Shopify) started monolithic and extracted micro-frontends when monolith friction became the actual bottleneck. Premature micro-frontend adoption is one of the costliest frontend mistakes.

---

## Implementing Micro-frontends with Module Federation

Module Federation is the primary technical mechanism to load remote mini-apps at runtime. It is a Webpack/Vite feature.

```javascript
// dashboard-app/webpack.config.js (the "remote" — is consumed by shell)
const { ModuleFederationPlugin } = require('webpack').container

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'dashboardApp',
      filename: 'remoteEntry.js',     // Entry point exposed to the shell
      exposes: {
        './Dashboard': './src/Dashboard', // Expose this component
      },
      shared: ['react', 'react-dom'],    // Share React to avoid duplicate bundles
    }),
  ],
}

// shell-app/webpack.config.js (the "host" — consumes the remote)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        dashboardApp: 'dashboardApp@http://localhost:3001/remoteEntry.js',
      },
      shared: ['react', 'react-dom'],
    }),
  ],
}

// shell-app/src/App.jsx — Lazy-load the remote component
import React, { Suspense, lazy } from 'react'
const RemoteDashboard = lazy(() => import('dashboardApp/Dashboard'))

export default function App() {
  return (
    <Suspense fallback={<div>Loading Dashboard...</div>}>
      <RemoteDashboard />
    </Suspense>
  )
}
```

→ See: [[Module federation]] for the full deep-dive.

---

## Related Notes
- [[Module federation]]
- [[Monorepo architecture]]
- [[Scalability in frontend systems]]
- [[SPA vs MPA]]
- [[High level architecture MOC]]
