---
created: 2026-04-10
tags: [moc, build, deployment, ci-cd]
related: [ [[Frontend system design MOC]], [[Developer experience (DX) MOC]] ]
status: active
---

# Build and deployment MOC

Build and deployment encompasses how frontend code is bundled, optimized, and shipped to users. A modern build pipeline is automated, fast, and reproducible.

## The Pipeline

```
Code Push → CI (lint, test, build) → Artifact → CD (deploy to edge/CDN)
```

## Notes in this Section
- [[Vite vs Webpack vs Turbopack]]: Comparing the major bundlers and their trade-offs.
- [[CI/CD for frontend]]: Automating testing, building, and deploying.
- [[Edge deployment and CDN]]: Pushing assets and compute as close to the user as possible.

## Key Concepts

| Concept | Purpose |
|---|---|
| **Bundling** | Combine many JS modules into optimized files |
| **Tree-shaking** | Remove unused code from the final bundle |
| **Code splitting** | Break bundles into lazy-loaded chunks |
| **Minification** | Remove whitespace, shorten variable names |
| **Source maps** | Map minified code back to original source for debugging |
| **Cache busting** | Content hashing to invalidate CDN caches on deploy |

---
## Related Topics
- [[Frontend system design MOC]]
- [[Developer experience (DX) MOC]]
- [[Code and bundle optimization]]
- [[Route based code splitting]]
