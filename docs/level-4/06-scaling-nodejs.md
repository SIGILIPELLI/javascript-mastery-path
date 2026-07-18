# 06 · Scaling Node.js Applications

Node.js runs JavaScript on a single thread. That's fine for I/O-bound work
(most web APIs), but it means one Node process can only use one CPU core —
scaling further requires clustering, caching, and load balancing, each
covered here with real code.

## Node is single-threaded — why that matters

A single Node process handles concurrent requests well because I/O
(database calls, HTTP requests) is non-blocking — but CPU-bound work (heavy
computation, image processing, big JSON parsing) blocks the entire event
loop, stalling every other in-flight request.

```javascript
// BAD — a CPU-heavy synchronous loop blocks ALL other requests
app.get("/report", (req, res) => {
  let total = 0;
  for (let i = 0; i < 10_000_000_000; i++) total += i; // blocks the event loop
  res.json({ total });
});
```

```javascript
// BETTER — offload CPU-heavy work to a worker thread, keep the event loop free
import { Worker } from "node:worker_threads";

app.get("/report", (req, res) => {
  const worker = new Worker("./report-worker.js");
  worker.once("message", (total) => res.json({ total }));
  worker.once("error", (error) => res.status(500).json({ error: error.message }));
});
```

## Clustering with Node's `cluster` module

`cluster` forks multiple copies of your Node process — one per CPU core —
each with its own event loop, sharing the same listening port. A crashed
worker doesn't take down the whole app.

```javascript
// cluster.js — entry point that forks one worker per CPU core
import cluster from "node:cluster";
import os from "node:os";
import process from "node:process";

const numCPUs = os.availableParallelism(); // or os.cpus().length

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running, forking ${numCPUs} workers`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker, code, signal) => {
    console.error(`Worker ${worker.process.pid} died (${signal || code}). Restarting...`);
    cluster.fork(); // keep the pool at full strength
  });
} else {
  // Each worker runs its own independent copy of the actual app
  await import("./server.js");
  console.log(`Worker ${process.pid} started`);
}
```

```javascript
// server.js — a normal Express app, unaware it's running inside a cluster worker
import express from "express";

const app = express();
app.get("/health", (req, res) => res.json({ pid: process.pid, status: "ok" }));
app.listen(3000);
```

Requests are load-balanced across workers by the OS/Node's internal round-robin
scheduler. In production, tools like **PM2** wrap this pattern with process
management, log aggregation, and zero-downtime reloads so you rarely hand-roll
`cluster` directly:

```bash
npm install -g pm2
pm2 start server.js -i max   # "-i max" = one worker per CPU core
pm2 reload server.js          # zero-downtime restart, one worker at a time
```

## Caching strategies

Caching avoids repeating expensive work (database queries, external API
calls, computation) for data that doesn't change on every request.

```javascript
// In-memory cache — fast, but per-process (each cluster worker has its own copy)
// and lost on restart. Fine for single-instance apps or short-lived data.
const cache = new Map();
const TTL_MS = 60_000;

async function getProduct(id) {
  const cached = cache.get(id);
  if (cached && Date.now() - cached.timestamp < TTL_MS) {
    return cached.data;
  }

  const data = await db.products.findById(id);
  cache.set(id, { data, timestamp: Date.now() });
  return data;
}
```

```javascript
// Redis cache — shared across all instances/processes, survives individual
// process restarts, supports TTL natively
import { createClient } from "redis";

const redis = createClient({ url: process.env.REDIS_URL });
await redis.connect();

async function getProductCached(id) {
  const cacheKey = `product:${id}`;
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  const data = await db.products.findById(id);
  await redis.set(cacheKey, JSON.stringify(data), { EX: 60 }); // expires in 60s
  return data;
}

// Cache invalidation — clear the cache entry whenever the underlying data changes
async function updateProduct(id, changes) {
  const updated = await db.products.update(id, changes);
  await redis.del(`product:${id}`); // "cache invalidation is one of the two hard things"
  return updated;
}
```

| Strategy | Where it lives | Survives restart? | Shared across instances? | Best for |
|---|---|---|---|---|
| In-memory (`Map`) | Inside one Node process | No | No | Single-instance apps, hot/small data |
| Redis / Memcached | External service | Yes | Yes | Multi-instance/clustered deployments |
| CDN edge cache | Edge network (Cloudflare, Fastly) | Yes | Yes (globally) | Static assets, public API responses |
| HTTP cache headers | Client/browser | Yes | No (per-client) | Reducing repeat requests from the same client |

## Load balancing

When you run multiple Node instances (via `cluster`, multiple containers, or
multiple machines), a load balancer distributes incoming traffic across them.

```nginx
# nginx.conf — a simple round-robin load balancer in front of 3 Node instances
upstream node_app {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;
    location / {
        proxy_pass http://node_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

In containerized/cloud deployments, this role is usually filled by a managed
load balancer (AWS ALB, GCP Load Balancer, Kubernetes `Service`) rather than a
hand-configured nginx box — but the concept (distribute requests, health-check
backends, remove unhealthy ones from rotation) is identical.

```javascript
// A health-check endpoint the load balancer polls to decide whether to route traffic here
app.get("/health", async (req, res) => {
  try {
    await db.ping(); // verify the dependency this instance actually needs
    res.status(200).json({ status: "ok" });
  } catch {
    res.status(503).json({ status: "unhealthy" }); // load balancer stops routing here
  }
});
```

## Horizontal vs. vertical scaling

| | Vertical scaling | Horizontal scaling |
|---|---|---|
| How | Bigger machine (more CPU/RAM) | More machines/processes |
| Node fit | Limited — single thread caps CPU use per process | Natural fit — `cluster`, containers, multiple instances |
| Downtime to scale | Often requires a restart | New instances added/removed without downtime |
| Ceiling | Hardware limits of one machine | Practically unbounded (with a data layer that can keep up) |

## Exercise

You have a Node API currently running as a single process that's pegged at
100% CPU on one core while 7 other cores sit idle. Using the `cluster`
snippet above as a starting point, write the entry-point code to run it
across all available cores, and describe (a sentence or two) how you'd add a
Redis-backed cache for a `GET /products/:id` endpoint that's hit far more
often than products change.
