---
created: 2026-04-07
tags: [security, csrf, cookies]
related: [ [[Frontend security MOC]], [[Session vs JWT authentication]], [[XSS protection]] ]
status: active
---

# CSRF protection

**Cross-Site Request Forgery (CSRF)** is an attack that tricks an authenticated user's browser into sending an unintended request to your server. Because the browser automatically sends cookies, a malicious site can trigger state-changing requests (transfers, password changes) on behalf of the logged-in user.

## How the Attack Works

```
1. User logs in to bank.com — receives a session cookie.
2. User visits evil.com in another tab (still logged in to bank).
3. evil.com's HTML contains:
   <img src="https://bank.com/transfer?to=attacker&amount=10000" />
4. The browser fetches that URL, automatically sending the session cookie.
5. bank.com receives a "valid" authenticated request and transfers the money.
```

---

## Prevention Strategies

### 1. `SameSite` Cookie Attribute (Modern Standard)
The most effective CSRF mitigation available today. Tells the browser when to send the cookie on cross-site requests.

| Value | Behavior | Use Case |
|---|---|---|
| `Strict` | Never sent on cross-site requests | Highest security; may break OAuth flows |
| `Lax` | Sent on top-level navigations (GET only) | Good default for most apps |
| `None` | Always sent (requires `Secure`) | Third-party embeds only |

```http
Set-Cookie: session=abc123; SameSite=Lax; Secure; HttpOnly
```

### 2. CSRF Tokens (Double Submit Pattern)
For older browsers or APIs that don't rely on cookies:

1. Server generates a unique, random **CSRF token** and sends it to the client (either in a cookie or embedded in the HTML).
2. Client includes the token in every state-changing request (POST/PUT/DELETE) as a custom header or request body field.
3. Server validates that the token in the header/body matches the expected value.

```tsx
// Client sends token in a custom header
async function updateProfile(data) {
  const csrfToken = getCookie('csrf-token'); // Cannot be read by attacker's site
  await fetch('/api/profile', {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken, // Attacker cannot set custom headers cross-origin
    },
    body: JSON.stringify(data),
  });
}
```

A cross-origin site cannot set custom request headers (blocked by the browser's same-origin policy), so even if the cookie is sent, the token header will be missing.

### 3. Origin / Referer Header Validation (Server-Side)
The server checks that the `Origin` or `Referer` header of the request matches the expected domain. This is a simple secondary check.

```ts
// Server-side check (Node.js/Express example)
app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (origin && origin !== 'https://yourapp.com') {
    return res.status(403).json({ error: 'CSRF detected' });
  }
  next();
});
```

---

## CSRF vs XSS

| | CSRF | XSS |
|---|---|---|
| **Goal** | Forge a user's request | Steal user data / run code |
| **Requires** | User is authenticated | Script injected into the page |
| **Fixed by** | `SameSite` cookies, CSRF tokens | Sanitization, CSP |

---

## Related Topics
- [[Frontend security MOC]]
- [[XSS protection]]
- [[Session vs JWT authentication]]
- [[Content Security Policy (CSP)]]
