# 07 · Real-Time Applications

HTTP request/response is a poor fit for features that need the server to push
data to clients instantly — chat, live notifications, collaborative editing,
live dashboards. WebSockets keep a persistent, two-way connection open so
either side can send a message at any time.

## Why not just poll?

Polling (repeatedly asking "anything new?") works but wastes requests and
adds latency equal to the poll interval. WebSockets open one long-lived
connection and push data the instant it's available.

```javascript
// Polling — simple, but wasteful and laggy
setInterval(async () => {
  const response = await fetch("/api/messages/latest");
  const messages = await response.json();
  renderMessages(messages);
}, 3000); // up to 3 seconds of lag, and a request every 3s even with no new data
```

## WebSockets with the `ws` package

`ws` is a minimal, fast WebSocket library for Node — no framework
abstractions, just raw send/receive.

```bash
npm install ws
```

```javascript
// server.js — a WebSocket server broadcasting a live counter to every client
import { WebSocketServer } from "ws";

const wss = new WebSocketServer({ port: 8080 });
let counter = 0;
const clients = new Set();

wss.on("connection", (socket) => {
  clients.add(socket);
  socket.send(JSON.stringify({ type: "counter", value: counter })); // send initial state

  socket.on("message", (raw) => {
    const message = JSON.parse(raw.toString());
    if (message.type === "increment") {
      counter += 1;
      broadcast({ type: "counter", value: counter }); // push to everyone, not just the sender
    }
  });

  socket.on("close", () => clients.delete(socket));
});

function broadcast(payload) {
  const data = JSON.stringify(payload);
  for (const client of clients) {
    if (client.readyState === client.OPEN) client.send(data);
  }
}
```

```javascript
// client.js — browser-side WebSocket client
const socket = new WebSocket("ws://localhost:8080");

socket.addEventListener("open", () => console.log("connected"));

socket.addEventListener("message", (event) => {
  const message = JSON.parse(event.data);
  if (message.type === "counter") {
    document.getElementById("counter").textContent = message.value;
  }
});

document.getElementById("increment-btn").addEventListener("click", () => {
  socket.send(JSON.stringify({ type: "increment" }));
});
```

## Socket.IO — WebSockets with fallbacks and rooms

Socket.IO builds on top of WebSockets (falling back to HTTP long-polling if
needed) and adds conveniences like automatic reconnection, event-based
messaging, and **rooms** for broadcasting to a subset of connected clients.

```bash
npm install socket.io
```

```javascript
// server.js — a simple live chat with rooms (one room per chat channel)
import { createServer } from "node:http";
import { Server } from "socket.io";
import express from "express";

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: "http://localhost:3000" } });

io.on("connection", (socket) => {
  console.log(`client connected: ${socket.id}`);

  socket.on("join-room", (roomId) => {
    socket.join(roomId);
  });

  socket.on("chat-message", ({ roomId, text, author }) => {
    const message = { text, author, timestamp: Date.now() };
    io.to(roomId).emit("chat-message", message); // only clients in this room receive it
  });

  socket.on("disconnect", () => {
    console.log(`client disconnected: ${socket.id}`);
  });
});

httpServer.listen(3000);
```

```javascript
// client — browser side, using the socket.io-client package
import { io } from "socket.io-client";

const socket = io("http://localhost:3000");

socket.emit("join-room", "general");

socket.on("chat-message", (message) => {
  appendMessageToUI(message);
});

function sendMessage(text) {
  socket.emit("chat-message", { roomId: "general", text, author: "Ada" });
}
```

## `ws` vs. Socket.IO

| Concern | `ws` | Socket.IO |
|---|---|---|
| Protocol | Raw WebSocket only | WebSocket + automatic HTTP long-polling fallback |
| Reconnection | Manual | Automatic |
| Rooms/namespaces | Manual (your own `Set`/`Map` bookkeeping) | Built-in |
| Message framing | You define the payload shape | Named events (`socket.emit("event", data)`) |
| Bundle/dependency size | Minimal | Larger (client + server libraries) |
| Best fit | Performance-critical, simple protocols, full control | Feature-rich apps wanting reconnection/rooms out of the box |

## Scaling WebSockets across multiple instances

A single Node process holds WebSocket connections in memory — if you run
multiple instances behind a load balancer (see [Module 6](06-scaling-nodejs.md)),
a message from a client on instance A won't reach a client connected to
instance B unless instances share state via something like Redis pub/sub.

```javascript
// Using Redis pub/sub so a broadcast reaches clients connected to ANY instance
import { createClient } from "redis";

const publisher = createClient({ url: process.env.REDIS_URL });
const subscriber = publisher.duplicate();
await publisher.connect();
await subscriber.connect();

await subscriber.subscribe("chat-messages", (message) => {
  broadcastToLocalClients(JSON.parse(message)); // fan out to this instance's own sockets
});

function publishMessage(message) {
  publisher.publish("chat-messages", JSON.stringify(message)); // every instance's subscriber gets it
}
```

Socket.IO ships an official `@socket.io/redis-adapter` that implements this
exact pattern for you, so rooms and broadcasts work correctly across a
horizontally-scaled cluster of servers.

## Exercise

Extend the `ws`-based live counter example above into a live "who's online"
feature: when a client connects, broadcast an updated count of currently
connected clients to everyone (including the new client), and broadcast an
updated (lower) count when a client disconnects. Sketch the message shape
you'd send, e.g. `{ type: "presence", count: 4 }`.
