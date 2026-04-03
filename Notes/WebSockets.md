---
created: 2026-04-03
tags: [real-time, websockets, network]
related: [ [[Real time communication MOC]], [[Server sent events (SSE)]] ]
status: active
---

# WebSockets

WebSockets provide a **full-duplex, persistent connection** between the client and server over a single TCP connection. Unlike HTTP, both sides can send messages to each other at any time without a new request-response cycle.

## How the Connection Works

1. The client sends an HTTP **Upgrade request** (the "handshake").
2. The server responds with `101 Switching Protocols`.
3. The HTTP connection is **upgraded to a WebSocket connection**.
4. Both sides can now send frames freely until one side closes the connection.

```
Client → Server: GET /chat HTTP/1.1, Upgrade: websocket
Server → Client: 101 Switching Protocols
[Connection established — full duplex from here]
Client → Server: "Hello!"
Server → Client: "Hi back!"
```

---

## Client-Side Implementation

```tsx
// hooks/useWebSocket.ts
import { useEffect, useRef, useState } from 'react';

export function useWebSocket(url: string) {
  const ws = useRef<WebSocket | null>(null);
  const [messages, setMessages] = useState<string[]>([]);

  useEffect(() => {
    ws.current = new WebSocket(url);

    ws.current.onopen = () => console.log('Connected');
    ws.current.onmessage = (event) => {
      setMessages((prev) => [...prev, event.data]);
    };
    ws.current.onerror = (error) => console.error('WS Error:', error);
    ws.current.onclose = () => console.log('Disconnected');

    return () => ws.current?.close(); // Cleanup on unmount
  }, [url]);

  const sendMessage = (msg: string) => ws.current?.send(msg);

  return { messages, sendMessage };
}
```

---

## Reconnection & Resilience

Network connections drop. Always implement automatic reconnection:

```ts
function connectWithRetry(url: string, delay = 1000) {
  const ws = new WebSocket(url);

  ws.onclose = () => {
    console.log(`Reconnecting in ${delay}ms...`);
    setTimeout(() => connectWithRetry(url, Math.min(delay * 2, 30000)), delay);
  };
}
```

This is called **Exponential Backoff** — it prevents flooding the server during an outage.

---

## Libraries

- **Socket.io**: Adds reconnection, rooms, namespaces, and fallback to polling. Has a corresponding server library.
- **Ably / Pusher**: Managed WebSocket services for when you don't want to manage your own server.

```ts
import { io } from 'socket.io-client';
const socket = io('http://localhost:3000');

socket.emit('chat:message', 'Hello!');
socket.on('chat:message', (msg) => console.log(msg));
```

---

## Pros & Cons

| Pros | Cons |
|---|---|
| True bi-directional communication | Persistent connection uses server resources |
| Low latency (no HTTP overhead per message) | Requires stateful server (complicates horizontal scaling) |
| Ideal for high-frequency updates | More complex than HTTP for simple use cases |
| Binary data support | Firewalls/proxies can sometimes block WS |

---

## Related Topics
- [[Real time communication MOC]]
- [[Server sent events (SSE)]]
- [[Polling and long polling]]
