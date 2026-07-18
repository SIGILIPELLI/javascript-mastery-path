# 01 · Full-Stack Architecture Patterns

At Master level, the hard problems stop being "how do I write this function"
and become "how do I structure a codebase so a team of engineers can change it
safely for years." This module covers the architectural decisions that shape
everything else in Level 4.

## Layered architecture

A layered (n-tier) architecture separates a full-stack JavaScript app into
distinct responsibilities, each depending only on the layer(s) beneath it.
This keeps business logic testable without a database or HTTP server.

```text
src/
  routes/          # HTTP layer — parses requests, calls services, formats responses
    users.routes.js
  controllers/      # Thin glue between routes and services
    users.controller.js
  services/         # Business logic — framework-agnostic, easy to unit test
    users.service.js
  repositories/      # Data access — the only layer that knows about SQL/ORM
    users.repository.js
  models/            # Domain types / schema shapes
    user.model.js
```

```javascript
// services/users.service.js — pure business logic, no HTTP or SQL here
export function createUsersService(usersRepository) {
  return {
    async registerUser({ email, password }) {
      if (!email.includes("@")) {
        throw new Error("invalid email");
      }
      const existing = await usersRepository.findByEmail(email);
      if (existing) {
        throw new Error("email already registered");
      }
      return usersRepository.create({ email, password });
    },
  };
}
```

```javascript
// controllers/users.controller.js — translates HTTP <-> service calls
export function createUsersController(usersService) {
  return {
    async register(req, res) {
      try {
        const user = await usersService.registerUser(req.body);
        res.status(201).json(user);
      } catch (error) {
        res.status(400).json({ error: error.message });
      }
    },
  };
}
```

Because `usersService` only depends on an abstract `usersRepository`
interface, you can swap Postgres for an in-memory fake in tests without
touching business logic — this is dependency injection in practice, not just
a buzzword.

## Monorepos vs. polyrepos

A **monorepo** holds multiple related packages (API, web app, shared types,
CLI tools) in one repository with one version-controlled history. A
**polyrepo** splits each into its own repository. Neither is universally
"right" — the decision depends on team size, deployment independence, and how
much code is genuinely shared.

| Concern | Monorepo | Polyrepo |
|---|---|---|
| Sharing code (types, utils) | Trivial — direct import | Needs a published package |
| Atomic cross-package changes | One commit, one PR | Coordinated PRs across repos |
| CI build time | Can grow large; needs caching/filtering | Each repo's CI stays small |
| Independent deploy cadence | Requires tooling discipline | Natural — each repo deploys alone |
| Onboarding | One clone, one setup | Multiple clones, multiple setups |
| Tooling maturity needed | Higher (Nx, Turborepo, workspaces) | Lower |

A typical Node/JS monorepo uses npm/pnpm/yarn **workspaces** to link local
packages without publishing them to a registry:

```json
// package.json (repo root)
{
  "name": "acme-platform",
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
}
```

```text
acme-platform/
  apps/
    api/            # Express/Fastify backend
    web/            # React/Next.js frontend
  packages/
    shared-types/    # TypeScript types shared by api and web
    ui/              # Shared component library
  package.json
  turbo.json          # or nx.json — build orchestration/caching
```

```javascript
// apps/api/package.json (dependency on an internal workspace package)
{
  "name": "@acme/api",
  "dependencies": {
    "@acme/shared-types": "workspace:*"
  }
}
```

`workspace:*` tells the package manager "resolve this from the local
workspace, not the npm registry" — changes to `shared-types` are picked up
immediately by anything that imports it, with no publish step.

## Structuring a full-stack app for maintainability

A few principles keep a full-stack JavaScript codebase navigable as it grows:

- **Feature folders over type folders at scale.** Grouping by feature
  (`features/billing/`, `features/orders/`) rather than by type
  (`controllers/`, `services/`) keeps related code together and makes deletion
  safe — removing a feature means removing one folder.
- **A shared "core" package for cross-cutting concerns** — logging, config
  loading, error classes — imported by every service so behavior stays
  consistent.
- **Explicit module boundaries.** Each feature module exports a small public
  API (usually via an `index.js`/`index.ts` barrel file) and treats its own
  internals as private, even within the same repo.

```javascript
// features/billing/index.js — the only file other features may import from
export { createInvoice } from "./billing.service.js";
export { InvoiceStatus } from "./billing.types.js";
// billing.repository.js and internal helpers are NOT re-exported —
// nothing outside this folder should reach into billing's internals.
```

```javascript
// features/orders/order.service.js
import { createInvoice } from "../billing/index.js"; // uses billing's public API only
```

## Choosing an architecture style

| Style | Best fit | Trade-off |
|---|---|---|
| Layered (n-tier) monolith | Small-to-mid teams, one deployable | Can become a "big ball of mud" without discipline |
| Modular monolith (feature folders) | Mid-size teams, single deploy, clear ownership | Requires enforcing boundaries (lint rules, code review) |
| Microservices | Large orgs, independent scaling/deploy | Operational overhead: networking, observability, data consistency |
| Serverless functions | Spiky/unpredictable load, small discrete tasks | Cold starts, vendor lock-in, harder local debugging |

Most teams should start with a modular monolith and only split into
microservices once a specific, measured pain (deploy coupling, scaling one
piece independently, team ownership boundaries) justifies the operational
cost — covered in depth in [Module 3](03-microservices-serverless.md).

## Exercise

You're joining a team maintaining a full-stack JavaScript app that currently
has one giant `server.js` file (2,000+ lines) mixing routes, SQL queries, and
business logic. Sketch a target folder structure (layered or feature-folder
style — your choice) you'd migrate it to, and write a short paragraph
explaining the order you'd migrate pieces in so the app keeps working at every
step (no "big bang rewrite").
