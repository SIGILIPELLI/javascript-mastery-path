# 09 · Observability

Once an app runs in production across multiple instances, you can't attach a
debugger and step through a bug. Observability — structured logs, metrics,
and health checks — is how you understand what a running system is actually
doing without interrupting it.

## Structured logging with `pino`

Structured logs are machine-parseable JSON (not free-form text), which lets
log aggregators (Datadog, CloudWatch, Elastic) filter and query them reliably.
`pino` is a very low-overhead structured logger for Node.

```bash
npm install pino pino-http
```

```javascript
// logger.js
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL || "info",
  formatters: {
    level: (label) => ({ level: label }), // log level as a string, not a number
  },
  timestamp: pino.stdTimeFunctions.isoTime,
});
```

```javascript
// server.js — attach request-scoped logging via pino-http
import express from "express";
import pinoHttp from "pino-http";
import { logger } from "./logger.js";

const app = express();
app.use(pinoHttp({ logger }));

app.get("/orders/:id", async (req, res) => {
  req.log.info({ orderId: req.params.id }, "fetching order"); // structured fields, not string concat

  try {
    const order = await ordersService.findById(req.params.id);
    if (!order) {
      req.log.warn({ orderId: req.params.id }, "order not found");
      return res.status(404).json({ error: "not found" });
    }
    res.json(order);
  } catch (error) {
    req.log.error({ err: error, orderId: req.params.id }, "failed to fetch order");
    res.status(500).json({ error: "internal error" });
  }
});
```

```json
// example log line pino emits — one JSON object per line, easy to grep/index
{"level":"info","time":"2026-07-18T10:22:31.000Z","orderId":"o-123","msg":"fetching order"}
```

## Structured logging with `winston`

`winston` is more configurable (multiple "transports" — files, HTTP, third-party
services — at once) at the cost of more overhead than `pino`.

```bash
npm install winston
```

```javascript
// logger.js
import winston from "winston";

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json() // structured JSON output, same goal as pino above
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "error.log", level: "error" }),
  ],
});

logger.info("order created", { orderId: "o-123", total: 42.5 });
logger.error("payment failed", { orderId: "o-123", reason: "card_declined" });
```

## `pino` vs. `winston`

| Concern | pino | winston |
|---|---|---|
| Performance | Very high (minimal per-call overhead) | Lower (more features cost cycles) |
| Output format | JSON by default | Configurable (JSON, plain text, custom) |
| Transports (file, HTTP, etc.) | Via separate `pino-transport` plugins | Built-in, many out of the box |
| Best fit | High-throughput services where log volume matters | Apps needing flexible multi-destination logging |

## Never log secrets

Redact sensitive fields before they ever reach a log line — logs often end up
in less access-controlled systems than your database.

```javascript
// A redaction list applied at the logger level (pino supports this natively)
const logger = pino({
  redact: ["req.headers.authorization", "*.password", "*.creditCard"],
});
```

## Health-check endpoints

A health-check endpoint tells orchestrators (Kubernetes, a load balancer, PM2)
whether an instance is ready to receive traffic — distinguish **liveness**
("is the process alive?") from **readiness** ("can it currently serve
requests?").

```javascript
// health.js
app.get("/health/live", (req, res) => {
  res.status(200).json({ status: "ok" }); // process is up — no dependency checks
});

app.get("/health/ready", async (req, res) => {
  const checks = {};
  let healthy = true;

  try {
    await db.query("SELECT 1");
    checks.database = "ok";
  } catch {
    checks.database = "unreachable";
    healthy = false;
  }

  try {
    await redis.ping();
    checks.cache = "ok";
  } catch {
    checks.cache = "unreachable";
    healthy = false;
  }

  res.status(healthy ? 200 : 503).json({ status: healthy ? "ready" : "not ready", checks });
});
```

## Basic metrics

Metrics are numeric time-series data (request counts, latency, error rates)
aggregated over time — cheaper to store and query at scale than raw logs, and
the basis for dashboards and alerts.

```bash
npm install prom-client
```

```javascript
// metrics.js — expose Prometheus-format metrics for scraping
import client from "prom-client";

const register = new client.Registry();
client.collectDefaultMetrics({ register }); // CPU, memory, event loop lag, etc.

const httpRequestDuration = new client.Histogram({
  name: "http_request_duration_seconds",
  help: "Duration of HTTP requests in seconds",
  labelNames: ["method", "route", "status_code"],
  registers: [register],
});

export function metricsMiddleware(req, res, next) {
  const end = httpRequestDuration.startTimer();
  res.on("finish", () => {
    end({ method: req.method, route: req.route?.path || "unknown", status_code: res.statusCode });
  });
  next();
}

app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});
```

## The three pillars, compared

| Pillar | Answers | Tooling examples |
|---|---|---|
| Logs | What exactly happened, in detail, on one request/event | pino, winston, CloudWatch Logs, Elastic |
| Metrics | How is the system behaving in aggregate, over time | Prometheus, Datadog, CloudWatch Metrics |
| Traces | How did one request flow across multiple services | OpenTelemetry, Jaeger, Datadog APM |

## Exercise

Add structured logging (with `pino`, redacting `password` and `authorization`
fields) plus a `/health/ready` endpoint to a small Express app that depends on
a Postgres database and a Redis cache. The readiness endpoint should return
`503` if either dependency is unreachable, and each check's individual status
should be visible in the JSON response body, as in the example above.
