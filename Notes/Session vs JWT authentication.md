---
created: 2026-04-02
tags: [authentication, security, jwt]
related: [ [[Authentication and authorization MOC]], [[Frontend security MOC]] ]
status: active
---

# Session vs JWT authentication

Two dominant approaches to maintaining a user's authenticated state across requests.

## 1. Session-based Authentication

### How it works
1. User logs in with credentials.
2. Server creates a **session record** in a database (or Redis) and returns a `session_id`.
3. The `session_id` is stored in an **httpOnly Cookie** on the client.
4. On every request, the browser sends the cookie automatically. The server looks up the `session_id` in the database to verify the user.

### Pros
- **Easy Invalidation**: Just delete the session from the DB to log the user out immediately.
- **Secure by Default**: The `httpOnly` cookie is immune to XSS attacks (JS cannot read it).

### Cons
- **Stateful**: The server must maintain and query a session store, creating a scalability challenge.
- **Sticky Sessions**: In multi-server environments, requests must hit the same server (unless using a shared Redis store).

---

## 2. JWT (JSON Web Token) Authentication

### How it works
1. User logs in. Server creates a cryptographically signed **JWT** (header + payload + signature).
2. The JWT is returned to the client and stored (in memory, localStorage, or a cookie).
3. The client sends the JWT in the `Authorization: Bearer <token>` header on every request.
4. The server verifies the signature — no database lookup needed.

### JWT Structure
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  <- Header (algo)
.eyJ1c2VySWQiOiIxMjMifQ                <- Payload (user data)
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV   <- Signature (verification)
```

### Pros
- **Stateless**: No session store needed; the server verifies the signature alone.
- **Scalable**: Works across multiple servers and microservices without shared state.
- **Portable**: Can be used across domains (useful for APIs).

### Cons
- **No Revocation**: A JWT is valid until it expires. You cannot instantly invalidate it (requires a blocklist).
- **Token Size**: JWTs are larger than a session ID cookie.
- **Storage Risk**: Storing in `localStorage` exposes it to XSS. Storing in `httpOnly` cookie reintroduces CSRF risk.

---

## Token Storage Trade-offs

| Strategy | XSS Safe | CSRF Safe | Notes |
|---|---|---|---|
| `localStorage` | ❌ No | ✅ Yes | Accessible to JS |
| `httpOnly Cookie` | ✅ Yes | ❌ No | Needs CSRF protection |
| In-Memory (React state) | ✅ Yes | ✅ Yes | Lost on page refresh |

**Best Practice**: Store access tokens in memory, use `httpOnly` cookies for refresh tokens.

---

## Related Topics
- [[Authentication and authorization MOC]]
- [[OAuth and social login]]
- [[Frontend security MOC]]
- [[CSRF protection]]
