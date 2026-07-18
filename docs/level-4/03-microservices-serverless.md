# 03 · Microservices & Serverless Patterns

Once a monolith genuinely outgrows a single deployable unit, two common paths
open up: splitting into independently-deployed **microservices**, or going
**serverless** and letting a cloud platform run individual functions on
demand. Both trade simplicity for scalability and independence — know the
cost before you pay it.

## What makes a good microservice boundary

The most common microservices mistake is splitting by technical layer
("auth service", "database service") instead of by **business capability**.
A good boundary aligns with a bounded context that a small team can own end
to end.

```text
Good boundaries (by business capability):
  orders-service      — owns order creation, status, history
  inventory-service    — owns stock levels, reservations
  notifications-service — owns email/SMS/push delivery

Bad boundaries (by technical layer):
  database-service     — every other service depends on this for all queries
  validation-service    — a shared library pretending to be a service
```

A useful test: can you deploy this service on its own, with its own
datastore, without another team's release blocking you? If not, it's not
really an independent service yet — it's a distributed monolith, which
combines the operational cost of microservices with the coupling of a
monolith.

## Communication between services

Services talk to each other synchronously (HTTP/gRPC — simple, but couples
availability) or asynchronously (message queue/event bus — decouples
availability, adds eventual consistency).

```javascript
// Synchronous: orders-service calls inventory-service directly over HTTP
async function reserveStock(orderId, items) {
  const response = await fetch("http://inventory-service/reservations", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ orderId, items }),
  });

  if (!response.ok) {
    throw new Error(`inventory-service rejected reservation: ${response.status}`);
  }

  return response.json();
}
```

```javascript
// Asynchronous: orders-service publishes an event; inventory-service reacts
// (using a generic pub/sub client — the same shape applies to
// SQS/SNS, RabbitMQ, Kafka, or Redis Streams)

// orders-service — publisher
async function createOrder(order) {
  await db.orders.insert(order);
  await eventBus.publish("order.created", { orderId: order.id, items: order.items });
  return order;
}

// inventory-service — subscriber, runs independently
eventBus.subscribe("order.created", async (event) => {
  await reserveInventory(event.items);
});
```

Asynchronous messaging means `orders-service` doesn't go down if
`inventory-service` is temporarily unavailable — the event waits in the queue
— but it also means you must design for eventual consistency and duplicate
delivery (make handlers **idempotent**).

## Serverless / Functions-as-a-Service

Serverless platforms (AWS Lambda, Azure Functions, Google Cloud Functions,
Cloudflare Workers) run your function on demand, scale to zero when idle, and
bill per invocation instead of per running server. You write a **handler**
function; the platform manages the runtime, scaling, and infrastructure.

```javascript
// handler.js — an AWS Lambda-style handler for API Gateway (Node.js runtime)
export const handler = async (event) => {
  try {
    const body = JSON.parse(event.body ?? "{}");

    if (!body.email) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: "email is required" }),
      };
    }

    const user = await createUser(body.email);

    return {
      statusCode: 201,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(user),
    };
  } catch (error) {
    console.error("handler failed", error); // structured logs go to CloudWatch
    return {
      statusCode: 500,
      body: JSON.stringify({ error: "internal error" }),
    };
  }
};
```

```yaml
# serverless.yml — a minimal Serverless Framework config for the handler above
service: users-api

provider:
  name: aws
  runtime: nodejs20.x
  region: us-east-1

functions:
  createUser:
    handler: handler.handler
    events:
      - httpApi:
          path: /users
          method: post
```

A key serverless discipline: handlers should be **stateless** and short-lived.
Don't hold in-memory caches or long-lived connections you expect to persist
across invocations — the platform may reuse the execution environment
("warm start") or spin up a brand-new one ("cold start") without notice.

```javascript
// A defensive pattern: lazily initialize expensive resources
// so they're reused on warm starts but safely recreated on cold starts
let dbConnection;

async function getConnection() {
  if (!dbConnection) {
    dbConnection = await connectToDatabase(process.env.DATABASE_URL);
  }
  return dbConnection;
}

export const handler = async (event) => {
  const db = await getConnection();
  // ... use db
};
```

## Microservices vs. serverless vs. monolith

| Concern | Monolith | Microservices | Serverless (FaaS) |
|---|---|---|---|
| Deployment unit | One app | Many independently-deployed services | Individual functions |
| Scaling | Whole app scales together | Each service scales independently | Automatic, per-function, to zero |
| Operational overhead | Low | High (networking, service discovery, observability) | Low-to-medium (managed by platform) |
| Cold start latency | None | None (services stay running) | Present, especially for infrequent functions |
| Best for | Small-mid teams, most products | Large orgs needing independent scaling/ownership | Spiky/event-driven workloads, glue code |
| Local dev/debugging | Easiest | Hardest (many services to run) | Moderate (emulators exist but imperfect) |

## Idempotency and retries

Distributed systems retry failed calls — your handlers must tolerate being
invoked more than once for the same logical request without corrupting data.

```javascript
// Idempotent handler using a client-supplied idempotency key
async function chargeCustomer(request) {
  const existingCharge = await db.charges.findByIdempotencyKey(request.idempotencyKey);
  if (existingCharge) {
    return existingCharge; // already processed — return the same result, don't double-charge
  }

  const charge = await paymentGateway.charge(request.amount, request.customerId);
  await db.charges.insert({ ...charge, idempotencyKey: request.idempotencyKey });
  return charge;
}
```

## Exercise

Design (no need to fully implement) a system for an e-commerce checkout flow
split into `orders-service`, `inventory-service`, and `payments-service`.
Sketch: (1) which service owns which data, (2) whether each interaction
between services should be synchronous or asynchronous and why, and (3) one
serverless function you'd add to this system (e.g. sending an order
confirmation email) and why a FaaS function is a better fit for it than a
long-running service.
