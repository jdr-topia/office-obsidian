---
created: 2026-04-02
tags: [moc, security, authentication]
related: [ [[Frontend system design MOC]], [[Frontend security MOC]] ]
status: active
---

# Authentication and authorization MOC

Authentication answers **"Who are you?"** and authorization answers **"What are you allowed to do?"** Both are critical to building secure, user-aware frontend applications.

## Authentication Strategies
- [[Session vs JWT authentication]]: Comparing server-managed sessions with stateless token-based auth.
- [[OAuth and social login]]: Delegating authentication to a trusted third party (Google, GitHub).
- [[Authentication with NextAuth]]: Implementing a full auth solution in a Next.js app.

## Authorization Patterns
- [[RBAC in frontend]]: Role-Based Access Control — restricting UI features by user role.
- [[Protected routes]]: Gating entire routes based on auth state.

## Key Concepts
- **Token Storage**: `localStorage` vs `httpOnly Cookie` — security implications.
- **CSRF vs XSS**: Understanding the attack vectors relevant to each storage strategy.
- **Token Refresh Flow**: Silently refreshing expired access tokens using refresh tokens.

---
## Related Topics
- [[Frontend system design MOC]]
- [[Frontend security MOC]]
- [[Routing strategy MOC]]
