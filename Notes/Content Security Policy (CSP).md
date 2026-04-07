---
created: 2026-04-07
tags: [security, csp, http-headers]
related: [ [[Frontend security MOC]], [[XSS protection]], [[CSRF protection]] ]
status: active
---

# Content Security Policy (CSP)

A **Content Security Policy** is an HTTP response header that tells the browser which sources of content (scripts, styles, images, fonts, etc.) are trusted. Even if an XSS attack injects a script into your page, the CSP can prevent that script from executing.

## How It Works

The server sends a `Content-Security-Policy` header. The browser parses it and blocks any resource that doesn't match the policy — even if the resource is already in the page's HTML.

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; style-src 'self' https://fonts.googleapis.com
```

---

## Key Directives

| Directive | Controls | Example |
|---|---|---|
| `default-src` | Fallback for all resource types | `'self'` |
| `script-src` | JavaScript sources | `'self' 'nonce-{nonce}'` |
| `style-src` | CSS sources | `'self' https://fonts.googleapis.com` |
| `img-src` | Image sources | `'self' data: https://cdn.example.com` |
| `connect-src` | Fetch / XHR / WebSocket endpoints | `'self' https://api.example.com` |
| `font-src` | Font sources | `https://fonts.gstatic.com` |
| `frame-ancestors` | Which sites can embed your app (prevents clickjacking) | `'none'` |
| `form-action` | Where forms can submit | `'self'` |
| `upgrade-insecure-requests` | Forces HTTP → HTTPS | (no value) |

## Source Expressions

| Expression | Meaning |
|---|---|
| `'self'` | Same origin only |
| `'none'` | Nothing allowed |
| `'unsafe-inline'` | Allows inline scripts/styles — **avoid** |
| `'unsafe-eval'` | Allows `eval()` — **avoid** |
| `'nonce-{value}'` | Allows a specific script with a matching nonce |
| `'strict-dynamic'` | Trusts scripts loaded by nonce-bearing scripts |
| `https://cdn.example.com` | A specific external host |

---

## Nonce-Based CSP (Recommended for Next.js)

`'unsafe-inline'` defeats the purpose of CSP. Instead, use a **nonce** — a one-time, cryptographically random token included in both the HTTP header and the `<script>` tag.

```ts
// middleware.ts — Generate a nonce for every request
import { NextResponse } from 'next/server';

export function middleware(request: Request) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');
  const csp = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
    style-src 'self' 'nonce-${nonce}';
    img-src 'self' blob: data:;
    font-src 'self';
  `.replace(/\s{2,}/g, ' ').trim();

  const response = NextResponse.next();
  response.headers.set('Content-Security-Policy', csp);
  response.headers.set('x-nonce', nonce); // Pass nonce to the app
  return response;
}
```

```tsx
// app/layout.tsx — Apply nonce to inline scripts
import { headers } from 'next/headers';

export default function RootLayout({ children }) {
  const nonce = headers().get('x-nonce');
  return (
    <html>
      <head>
        <script nonce={nonce} src="/analytics.js" />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

---

## Report-Only Mode

Use `Content-Security-Policy-Report-Only` to test a CSP without enforcing it. Violations are reported to your endpoint but not blocked.

```http
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-violations
```

This lets you audit what would be blocked before rolling out a strict CSP to production.

---

## Related Topics
- [[Frontend security MOC]]
- [[XSS protection]]
- [[CSRF protection]]
