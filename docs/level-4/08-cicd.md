# 08 · CI/CD for JavaScript Projects

Continuous Integration (CI) automatically checks every change (lint, test,
build) before it merges. Continuous Deployment/Delivery (CD) automatically
ships passing changes toward production. Together they turn "did I break
anything?" from a manual, error-prone question into an automated gate.

## CI vs. CD

| | Continuous Integration | Continuous Delivery | Continuous Deployment |
|---|---|---|---|
| What runs automatically | Lint, tests, build on every push/PR | CI, plus packaging a release-ready artifact | CI, plus automatic deploy to production |
| Human step remaining | Merge the PR | Click "deploy" | None — merging IS deploying |
| Typical trigger | Push, pull request | Merge to main | Merge to main |

Most teams land on continuous delivery for production deploys (a human still
approves the final push) and full continuous deployment for staging/preview
environments.

## A GitHub Actions workflow: lint → test → build → deploy

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [18, 20, 22] # verify against multiple supported Node versions
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm
      - run: npm ci
      - run: npm test -- --coverage
      - uses: codecov/codecov-action@v4 # optional — publish coverage reports
        with:
          fail_ci_if_error: false

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' # only deploy from main, never from PR branches
    environment: production               # requires manual approval if configured on the repo
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      - name: Deploy
        run: ./scripts/deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

A few details worth internalizing: `needs:` chains jobs so `test` won't run
if `lint` fails, saving CI minutes; the `matrix` strategy runs the same job
against several Node versions in parallel; and `if: github.ref == ...` keeps
deployment from firing on every pull request.

## Caching dependencies for faster CI

```yaml
# Already handled above by `cache: npm` in setup-node, but the manual
# equivalent (useful for non-npm caches, e.g. Playwright browsers) looks like:
- name: Cache Playwright browsers
  uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
```

## Required status checks — making CI a real gate

A CI workflow only prevents bad merges if the repository is configured to
require it. On GitHub: **Settings → Branches → Branch protection rules** →
require the `lint`, `test`, and `build` jobs to pass before a PR can merge,
and require branches to be up to date before merging.

## Deployment strategies

| Strategy | How it works | Rollback speed |
|---|---|---|
| Recreate | Stop old version, start new version | Slow (downtime during switch) |
| Rolling | Replace instances one at a time | Moderate |
| Blue-green | Run two full environments, switch traffic at once | Fast (switch traffic back) |
| Canary | Route a small % of traffic to the new version first | Fast (pull the canary, no wider impact) |

```yaml
# A canary-style deploy step using a hypothetical CLI —
# the pattern (small % first, then promote) generalizes across platforms
- name: Deploy canary (5% of traffic)
  run: ./scripts/deploy.sh --strategy canary --weight 5

- name: Wait and check error rate
  run: ./scripts/check-error-rate.sh --threshold 1%

- name: Promote to 100%
  run: ./scripts/deploy.sh --strategy canary --weight 100
```

## Secrets management

Never commit secrets (API keys, database URLs, deploy tokens) to the
repository — use the CI platform's secret store, injected as environment
variables at run time.

```yaml
# GitHub Actions — secrets configured under Settings → Secrets and variables
- name: Deploy
  run: ./scripts/deploy.sh
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

```bash
# .gitignore — make it structurally hard to commit secrets by accident
.env
.env.local
*.pem
```

## Exercise

A team's CI pipeline currently only runs `npm test` on every push, taking 12
minutes because it reinstalls dependencies from scratch every time and runs
lint, unit tests, and E2E tests all in one serial job. Redesign the workflow
(as YAML, using the structure above) into separate `lint`, `test`, and `e2e`
jobs that run in parallel where possible, with dependency caching enabled,
and explain in one sentence which job(s) should block a PR merge versus which
could be advisory-only.
