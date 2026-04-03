---
created: 2026-04-03
tags: [moc, real-time, websockets, sse]
related: [ [[Frontend system design MOC]], [[Data fetching strategy MOC]] ]
status: active
---

# Real time communication MOC

Real-time communication enables the server to push data to the client without the client explicitly requesting it. It powers live features like chat, notifications, collaborative editing, and live dashboards.

## Approaches Compared

| Approach | Direction | Protocol | Use Case |
|---|---|---|---|
| **Polling** | Client → Server (repeated) | HTTP | Simple, infrequent updates |
| **Long Polling** | Client → Server (held open) | HTTP | Low-frequency push on legacy infra |
| **Server-Sent Events (SSE)** | Server → Client (one-way) | HTTP | Live feeds, notifications |
| **WebSockets** | Bi-directional | TCP (WS/WSS) | Chat, gaming, collaboration |

## Notes in this Section
- [[WebSockets]]: Full-duplex bi-directional communication.
- [[Server sent events (SSE)]]: Efficient one-way streaming over HTTP.
- [[Polling and long polling]]: The simplest approaches to simulate real-time.

## Decision Guide

```
Need bi-directional? (user sends AND receives data)
  └─ Yes → WebSockets
  └─ No  → Does the server need to push to many clients efficiently?
              └─ Yes → SSE
              └─ No  → Polling / Long Polling
```

---
## Related Topics
- [[Frontend system design MOC]]
- [[Data fetching strategy MOC]]
- [[Caching strategies MOC]]
- [[Offline support MOC]]
