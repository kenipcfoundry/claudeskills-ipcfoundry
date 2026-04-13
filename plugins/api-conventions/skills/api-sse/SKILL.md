---
name: api-sse
description: SSE real-time update pattern in the backend repository using the displayBus singleton. Covers server-side event broadcasting and how services trigger updates after mutations.
---

# API Server-Sent Events (SSE)

## displayBus Singleton

The `displayBus` in `src/sse/displayBus.js` is an EventEmitter-based singleton that manages SSE client subscriptions and broadcasts updates:

```javascript
// displayBus.js
import { EventEmitter } from "events";

class DisplayBus extends EventEmitter {
  constructor() {
    super();
    this.clients = new Map();
  }

  subscribe(clientId, res, locationId) {
    this.clients.set(clientId, { res, locationId });
    res.on("close", () => this.clients.delete(clientId));
  }

  broadcast(locationId) {
    for (const [, client] of this.clients) {
      if (locationId && client.locationId && client.locationId !== locationId) continue;
      client.res.write(`data: ${JSON.stringify({ type: "update", locationId })}\n\n`);
    }
  }
}

export const displayBus = new DisplayBus();
```

## SSE Endpoint (server.js)

```javascript
app.get("/sse/displays", authenticateToken(true), (req, res) => {
  const locationId = Number(req.query.locationId) || null;
  const clientId = `${Date.now()}-${Math.random().toString(36).slice(2)}`;

  res.writeHead(200, {
    "Content-Type": "text/event-stream",
    "Cache-Control": "no-cache",
    Connection: "keep-alive",
  });

  res.write(`data: ${JSON.stringify({ type: "connected", clientId })}\n\n`);

  const heartbeat = setInterval(() => res.write(": heartbeat\n\n"), 30000);
  displayBus.subscribe(clientId, res, locationId);
  req.on("close", () => clearInterval(heartbeat));
});
```

## Broadcasting from Services

Call `displayBus.broadcast()` **after** the transaction commits, not inside:

```javascript
import { displayBus } from "../sse/displayBus.js";

export const updateItem = async (payload) => {
  let locationId;
  await prisma.$transaction(async (tx) => {
    // ... mutations
    locationId = result.locationId;
  });

  // Broadcast AFTER commit
  displayBus.broadcast(locationId);   // specific location
  displayBus.broadcast(null);         // all locations
};
```

## Rules

- `broadcast(locationId)` sends to clients subscribed to that location
- `broadcast(null)` sends to ALL connected clients
- Always broadcast after the transaction commits, never inside
- 30-second heartbeat keeps connections alive
- Clients auto-cleanup on disconnect
