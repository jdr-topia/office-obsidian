---
created: 2026-04-02
tags: [moc, error-handling, resilience]
related: [ [[Frontend system design MOC]] ]
status: active
---

# Error handling MOC

A resilient frontend is one that degrades gracefully. Error handling is not just about catching exceptions — it is about communicating failure clearly and recovering without breaking the entire application.

## Core Concepts
- [[Error boundaries in React]]: Catching render-time errors and preventing full app crashes.
- [[API error handling]]: Strategies for handling network failures, 4xx/5xx responses.
- [[Fallback UI patterns]]: What to show the user when something goes wrong.

## Layers of Error Handling

### 1. Component Layer
- React Error Boundaries catch errors in the component tree.
- Graceful fallback UIs instead of blank/broken screens.

### 2. Network / API Layer
- Handling HTTP status codes (`400`, `401`, `403`, `404`, `500`).
- Retry logic for transient failures (network timeouts).
- User-facing error messages and toast notifications.

### 3. Global / Monitoring Layer
- Centralized logging with tools like **Sentry** or **Datadog**.
- Unhandled promise rejection listeners (`window.addEventListener('unhandledrejection', ...)`).
- See [[Observability and monitoring MOC]].

---
## Related Topics
- [[Frontend system design MOC]]
- [[Observability and monitoring MOC]]
- [[Server state management]]
