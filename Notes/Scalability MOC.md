---
created: 2026-04-10
tags: [moc, scalability, monorepo, architecture]
related: [ [[Frontend system design MOC]], [[High level architecture MOC]] ]
status: active
---

# Scalability MOC

Frontend scalability addresses how your codebase, team, and infrastructure cope as the product grows — more features, more engineers, more traffic.

## Dimensions of Frontend Scalability

| Dimension | Challenge | Solution |
|---|---|---|
| **Codebase** | Too large for one repo | [[Monorepo architecture]] |
| **Runtime** | Scaling independently deployed UI pieces | [[Module federation]] |
| **Team** | Shared UI across multiple teams | [[Design systems and shared libraries]] |
| **Traffic** | Serving millions of users fast | [[Edge deployment and CDN]], [[CDN usage in frontend]] |

## Notes in this Section
- [[Monorepo architecture]]: Managing multiple apps and packages in one repository.
- [[Module federation]]: Runtime code sharing between independently deployed apps.
- [[Design systems and shared libraries]]: Shared UI contracts across teams.

---
## Related Topics
- [[Frontend system design MOC]]
- [[High level architecture MOC]]
- [[Monolith vs micro-frontend]]
- [[Build and deployment MOC]]
