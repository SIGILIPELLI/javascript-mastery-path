# 05 · Testing at Scale

Unit tests (Level 1-3) verify individual pieces in isolation. At Master
level, production systems also need **end-to-end (E2E) tests** that drive the
real application the way a user would — through the browser or the full HTTP
stack — plus a CI pipeline that runs all of it automatically.

## The testing pyramid, revisited

| Layer | What it checks | Speed | Typical tools |
|---|---|---|---|
| Unit | A single function/class in isolation | Milliseconds | Jest, Vitest |
| Integration | Several units together (e.g. service + real DB) | Seconds | Jest + Testcontainers, supertest |
| End-to-end (E2E) | Full user flow through a real browser/UI | Seconds-to-minutes | Playwright, Cypress |

You want many unit tests, fewer integration tests, and a small number of
high-value E2E tests covering critical user journeys (login, checkout,
core workflows) — not every possible click path, which would be slow and
brittle.

## E2E testing with Playwright

Playwright drives a real browser (Chromium, Firefox, WebKit), clicking,
typing, and asserting on rendered output — catching bugs unit tests structurally
cannot see (CSS hiding a button, a broken API integration, a JS error only in
Safari).

```bash
npm install -D @playwright/test
npx playwright install # downloads browser binaries
```

```javascript
// tests/e2e/login.spec.js
import { test, expect } from "@playwright/test";

test("user can log in and see their dashboard", async ({ page }) => {
  await page.goto("http://localhost:3000/login");

  await page.fill('input[name="email"]', "ada@example.com");
  await page.fill('input[name="password"]', "correct-horse-battery-staple");
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL("http://localhost:3000/dashboard");
  await expect(page.locator("h1")).toHaveText("Welcome back, Ada");
});

test("shows an error for invalid credentials", async ({ page }) => {
  await page.goto("http://localhost:3000/login");

  await page.fill('input[name="email"]', "ada@example.com");
  await page.fill('input[name="password"]', "wrong-password");
  await page.click('button[type="submit"]');

  await expect(page.locator('[role="alert"]')).toHaveText(/invalid email or password/i);
});
```

```javascript
// playwright.config.js
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  timeout: 30_000,
  retries: process.env.CI ? 2 : 0, // flaky E2E tests get automatic retries in CI
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry", // captures a debuggable trace when a retry happens
  },
  webServer: {
    command: "npm run start", // Playwright boots the app before running tests
    port: 3000,
    reuseExistingServer: !process.env.CI,
  },
});
```

## E2E testing with Cypress

Cypress runs inside the browser itself (rather than driving it externally
like Playwright), which gives very fast feedback and time-travel debugging in
its test runner UI.

```bash
npm install -D cypress
```

```javascript
// cypress/e2e/checkout.cy.js
describe("checkout flow", () => {
  it("completes a purchase with a valid card", () => {
    cy.visit("/cart");
    cy.get('[data-testid="checkout-button"]').click();

    cy.get('input[name="cardNumber"]').type("4242424242424242");
    cy.get('input[name="expiry"]').type("12/30");
    cy.get('input[name="cvc"]').type("123");
    cy.get('button[type="submit"]').click();

    cy.contains("Order confirmed").should("be.visible");
    cy.url().should("include", "/order-confirmation");
  });
});
```

```javascript
// cypress.config.js
import { defineConfig } from "cypress";

export default defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
    retries: { runMode: 2, openMode: 0 }, // retry in headless CI runs only
  },
});
```

## Playwright vs. Cypress

| Concern | Playwright | Cypress |
|---|---|---|
| Browser engines | Chromium, Firefox, WebKit | Chromium-family (+ experimental Firefox/WebKit) |
| Execution model | Drives browser via CDP, out-of-process | Runs inside the browser |
| Multi-tab / multi-origin | Native support | Limited, historically harder |
| Parallelization | Built-in test sharding | Built-in (paid dashboard for orchestration) |
| Auto-wait for elements | Yes | Yes |
| Best fit | Cross-browser coverage, multi-tab flows | Fast local dev feedback loop, strong DX |

## Integration testing an API with supertest

Between unit and full-browser E2E sits API-level integration testing —
exercising real HTTP routes without a browser.

```bash
npm install -D supertest
```

```javascript
// tests/integration/users.test.js
import request from "supertest";
import { app } from "../../src/app.js";

describe("POST /users", () => {
  it("creates a user and returns 201", async () => {
    const response = await request(app)
      .post("/users")
      .send({ email: "ada@example.com", password: "s3cure-pass" });

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({ email: "ada@example.com" });
    expect(response.body.password).toBeUndefined(); // never leak password hashes
  });

  it("rejects a duplicate email with 409", async () => {
    await request(app).post("/users").send({ email: "dup@example.com", password: "x" });
    const response = await request(app).post("/users").send({ email: "dup@example.com", password: "y" });

    expect(response.status).toBe(409);
  });
});
```

## Wiring E2E tests into CI

E2E tests are slower and flakier than unit tests, so they typically run in a
separate CI job/stage — often only on the main branch or nightly, not on
every single commit, to keep feedback loops fast.

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]
  pull_request:

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npx playwright test
      - uses: actions/upload-artifact@v4 # keep failure traces for debugging
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Reducing flakiness

- Prefer `await expect(locator).toHaveText(...)` (auto-retrying assertions)
  over manual `sleep()` calls.
- Use `data-testid` attributes for selectors instead of CSS classes that
  change with styling.
- Reset test data (a fresh database/seed) before each test run instead of
  relying on tests running in a specific order.
- Isolate tests from real third-party services (payment gateways, email
  providers) behind fakes/sandboxes so external outages don't fail your CI.

## Exercise

Write a Playwright test for a "forgot password" flow: visiting `/forgot-password`,
submitting an email, and asserting a confirmation message appears (e.g.
"If that email exists, a reset link has been sent"). Then write the matching
GitHub Actions job snippet that runs this spec on every pull request, using
`retries: 1` to absorb one flaky run without failing the build outright.
