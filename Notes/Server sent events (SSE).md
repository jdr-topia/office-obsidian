---
created: 2026-04-03
tags: [real-time, sse, http, streaming]
related: [ [[Real time communication MOC]], [[WebSockets]], [[Streaming SSR]] ]
status: active
---

# Server sent events (SSE)

**Server-Sent Events (SSE)** is a standard HTTP mechanism for the server to push a stream of events to the client over a **single, long-lived HTTP connection**. Unlike WebSockets, it is strictly **one-way** (server → client).

## How It Works

The client makes a standard `GET` request. The server keeps the connection open and streams `text/event-stream` formatted data for as long as needed.

```
Client: GET /api/live-feed HTTP/1.1
Server: HTTP/1.1 200 OK
        Content-Type: text/event-stream

        data: {"price": 150.23}

        data: {"price": 150.81}

        event: alert
        data: {"message": "Flash sale started!"}
```

---

## Client-Side Implementation

Built into all modern browsers via the `EventSource` API — **no library needed**.

```tsx
// hooks/useSSE.ts
import { useEffect, useState } from 'react';

export function useSSE<T>(url: string) {
  const [data, setData] = useState<T | null>(null);

  useEffect(() => {
    const eventSource = new EventSource(url);

    eventSource.onmessage = (event) => {
      setData(JSON.parse(event.data));
    };

    eventSource.addEventListener('alert', (event) => {
      console.log('Custom event:', JSON.parse(event.data));
    });

    eventSource.onerror = () => {
      eventSource.close(); // Close on error; browser will auto-reconnect by default
    };

    return () => eventSource.close(); // Cleanup
  }, [url]);

  return data;
}
```

---

## Next.js Route Handler (Server Side)

```ts
// app/api/live-feed/route.ts
export async function GET() {
  const stream = new ReadableStream({
    start(controller) {
      const encoder = new TextEncoder();

      const interval = setInterval(() => {
        const payload = `data: ${JSON.stringify({ time: Date.now() })}\n\n`;
        controller.enqueue(encoder.encode(payload));
      }, 1000);

      // Stop after 30 seconds
      setTimeout(() => {
        clearInterval(interval);
        controller.close();
      }, 30000);
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

---

## SSE vs WebSockets

| Feature | SSE | WebSockets |
|---|---|---|
| Direction | Server → Client only | Bi-directional |
| Protocol | HTTP/HTTPS | WS/WSS (TCP) |
| Browser Support | Native `EventSource` API | Native `WebSocket` API |
| Auto-Reconnect | ✅ Built-in | ❌ Manual |
| HTTP/2 Multiplexing | ✅ Yes | ❌ No (separate connection) |
| Firewalls/Proxies | ✅ Usually passes | Sometimes blocked |
| Best For | Feeds, notifications, progress | Chat, gaming, collaboration |

---

## Related Topics
- [[Real time communication MOC]]
- [[WebSockets]]
- [[Polling and long polling]]
- [[Streaming SSR]]
