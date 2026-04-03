---
created: 2026-04-03
tags: [real-time, polling, http, network]
related: [ [[Real time communication MOC]], [[WebSockets]], [[Server sent events (SSE)]] ]
status: active
---

# Polling and long polling

Polling is the simplest approach to simulate real-time updates. The client repeatedly asks the server "is there anything new?" at a fixed interval.

---

## 1. Short Polling (Regular Polling)

The client sends a request every N seconds regardless of whether new data exists.

```ts
// hooks/usePolling.ts
import { useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';

export function usePolling<T>(queryKey: string[], queryFn: () => Promise<T>, intervalMs = 5000) {
  return useQuery({
    queryKey,
    queryFn,
    refetchInterval: intervalMs,        // Refetch every 5 seconds
    refetchIntervalInBackground: false, // Stop polling when tab is not focused
  });
}

// Usage:
const { data } = usePolling(['notifications'], fetchNotifications, 10000);
```

### Pros
- Dead simple to implement.
- Works everywhere — no special server setup.

### Cons
- **Wasteful**: Most responses return "nothing new." Bandwidth and server CPU are consumed unnecessarily.
- **Latency**: A new event appears at most `interval` seconds late.

---

## 2. Long Polling

An improvement on polling. The client makes a request, and the server **holds it open** until new data is available (or a timeout occurs). As soon as the server responds, the client immediately makes a new request.

```
Client → Server: GET /updates (waiting...)
[Server holds the connection for up to 30 seconds]
[New data arrives on the server]
Server → Client: 200 OK { "notification": "..." }
Client → Server: GET /updates (immediately re-requests)
```

### Implementation (Conceptual)

```ts
async function longPoll() {
  try {
    // Server will respond when data is available (or after timeout)
    const response = await fetch('/api/updates?timeout=30');
    const data = await response.json();
    processUpdate(data);
  } catch (err) {
    await new Promise(resolve => setTimeout(resolve, 1000)); // Short delay on error
  } finally {
    longPoll(); // Immediately re-poll
  }
}

longPoll(); // Start
```

### Pros
- Lower latency than short polling.
- Events are delivered almost immediately when they occur.

### Cons
- Ties up server threads waiting for responses.
- More complex to implement on the server than short polling.
- Still incurs HTTP overhead on every cycle.

---

## When to Use Polling vs SSE vs WebSockets

| Scenario | Best Choice |
|---|---|
| Dashboard refresh every 30s | Short Polling |
| Live notification badge | Long Polling or SSE |
| Live comment feed | SSE |
| Chat application | WebSockets |
| Collaborative document editing | WebSockets |
| Querying a legacy API (no WS/SSE support) | Long Polling |

---

## Related Topics
- [[Real time communication MOC]]
- [[WebSockets]]
- [[Server sent events (SSE)]]
- [[Optimistic updates and retry strategies]]
