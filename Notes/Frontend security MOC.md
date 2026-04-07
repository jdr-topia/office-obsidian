---
created: 2026-04-07
tags: [moc, security, frontend]
related: [ [[Frontend system design MOC]], [[Authentication and authorization MOC]] ]
status: active
---

# Frontend security MOC

Frontend security focuses on protecting users and the application from attacks that exploit the browser environment. While the backend is the true security boundary, the frontend is the first line of defense.

## The Core Threat Model

| Attack | Vector | Mitigation |
|---|---|---|
| **XSS** | Injecting malicious scripts into the page | [[XSS protection]] |
| **CSRF** | Tricking authenticated users into unintended requests | [[CSRF protection]] |
| **Clickjacking** | Embedding your app in an iframe to steal clicks | CSP `frame-ancestors` |
| **Sensitive Data Exposure** | Leaking tokens, keys, or PII | Secure storage, HTTPS |
| **Dependency Attacks** | Malicious npm packages in your supply chain | Auditing, `npm audit` |

## Notes in this Section
- [[XSS protection]]: Preventing injection of malicious scripts.
- [[CSRF protection]]: Preventing cross-site request forgery.
- [[Content Security Policy (CSP)]]: A browser-enforced allowlist for resources.

---
## Related Topics
- [[Frontend system design MOC]]
- [[Authentication and authorization MOC]]
- [[Session vs JWT authentication]]
