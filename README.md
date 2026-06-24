# FakeStoreAPI — API Testing (Postman + Newman)

End-to-end API test suite for [FakeStoreAPI](https://fakestoreapi.com/docs), built with **Postman** and executed in CI with **Newman** + **GitHub Actions**.

[![API Tests](https://github.com/SamantaRubio/PostmanNewman/actions/workflows/api-tests.yml/badge.svg)](https://github.com/SamantaRubio/PostmanNewman/actions/workflows/api-tests.yml)

## What this project demonstrates

- **Full CRUD coverage** across the `Products`, `Carts`, `Users` and `Auth` resources.
- **JSON Schema validation** with [AJV](https://ajv.js.org/) — assertions go beyond status codes and verify the response *contract*.
- **Variable chaining** — values captured in one request (auth token, product/cart/user IDs, categories) feed later requests.
- **Negative testing** — unknown IDs and invalid routes.
- **Data-driven testing** — `POST /products` runs once per row in `data/products.json`.
- **Collection-level assertions** — every request is checked for response time and absence of 5xx errors.
- **CI/CD** — runs on every push/PR, daily on a schedule, and publishes HTML + JUnit reports as artifacts.

## Project structure

```
.
├── .github/workflows/api-tests.yml          # GitHub Actions pipeline
├── collections/
│   └── FakeStoreAPI.postman_collection.json # The test collection
├── environments/
│   └── fakestore.postman_environment.json   # Environment variables (baseUrl, token)
├── data/
│   └── products.json                        # Data-driven test inputs
├── reports/                                 # Generated reports (gitignored)
├── package.json
└── README.md
```

## Prerequisites

- [Node.js](https://nodejs.org/) 18+
- npm (ships with Node)

## Setup

```bash
npm install
```

## Running the tests

| Command | Description |
|---|---|
| `npm test` | Run the full collection (CLI output). |
| `npm run test:data` | Run with the data file (data-driven `POST /products`). |
| `npm run test:html` | Run and export an HTML report to `reports/report.html`. |
| `npm run test:ci` | Run and export HTML + JUnit reports (used by CI). |

Example:

```bash
npm run test:html
open reports/report.html   # macOS
```

## Test coverage

| Folder | Requests |
|---|---|
| **Auth** | Login (valid) · Login (invalid → 401) |
| **Products** | Get all · Get by id · Limit · Sort · Categories · By category · Create · Update (PUT) · Update (PATCH) · Delete |
| **Carts** | Get all · Get by id · Get by user · Create · Delete |
| **Users** | Get all · Get by id · Create · Update · Delete |
| **Negative cases** | Unknown product id · Unknown route (404) |

## Continuous Integration (CI/CD)

Tests run automatically on **GitHub Actions** — no local setup required to see them pass. The pipeline is defined in [`.github/workflows/api-tests.yml`](.github/workflows/api-tests.yml).

**Triggers:**

| Event | When it runs |
|---|---|
| `push` | On every push to `main` |
| `pull_request` | On every PR targeting `main` |
| `schedule` | Daily at 08:00 UTC — catches regressions in the live API |
| `workflow_dispatch` | Manually, from the Actions tab |

**What the pipeline does:**

1. Checks out the repository.
2. Sets up Node.js 20 with npm cache.
3. Installs dependencies with `npm ci`.
4. Runs the collection via `npm run test:ci` (CLI + HTML + JUnit reporters).
5. Uploads `reports/` (HTML + JUnit) as a downloadable build artifact, kept for 30 days.
6. Publishes a test-results summary from the JUnit output.

The build **fails if any assertion fails**, so the status badge at the top of this README reflects the real state of the API contract. To download a report, open the latest run under the repo's **Actions** tab and grab the `newman-reports` artifact.

## Notes on FakeStoreAPI

FakeStoreAPI is a **mock backend**: `POST`/`PUT`/`PATCH`/`DELETE` requests return a *simulated* response (an echo of the payload plus an `id`) but **do not persist** any data. The tests assert against the simulated response shape rather than expecting the data to be retrievable afterwards — this mirrors how you would test a service whose write side-effects are out of scope.

## Author

**Samanta Rubio** — QA Automation Engineer
