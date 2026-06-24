# Restful Booker — API Testing (Postman + Newman)

End-to-end API test suite for [Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html), built with **Postman** and executed in CI with **Newman** + **GitHub Actions**.

[![API Tests](https://github.com/SamantaRubio/PostmanNewman/actions/workflows/api-tests.yml/badge.svg)](https://github.com/SamantaRubio/PostmanNewman/actions/workflows/api-tests.yml)

## What this project demonstrates

- **Real CRUD lifecycle** — a booking is created, read back, updated (PUT), partially updated (PATCH), deleted, and then confirmed gone (404). Unlike echo/mock APIs, Restful Booker **persists data**, so the tests verify actual state changes.
- **Token-based authentication** — a token is obtained from `POST /auth` and chained into the protected `PUT` / `PATCH` / `DELETE` requests via a `Cookie` header.
- **JSON Schema validation** with [AJV](https://ajv.js.org/) — assertions go beyond status codes and verify the response *contract*.
- **Variable chaining** — the auth token and the new booking id flow from one request to the next.
- **Negative testing** — bad credentials, unknown id (404), and an unauthorized update (403).
- **Data-driven testing** — the create/read/update/delete flow runs once per row in `data/bookings.json`.
- **Collection-level assertions** — every request is checked for response time and absence of 5xx errors.

## Project structure

```
.
├── .github/workflows/api-tests.yml             # GitHub Actions pipeline
├── collections/
│   └── RestfulBooker.postman_collection.json   # The test collection
├── environments/
│   └── restfulbooker.postman_environment.json  # Environment variables (baseUrl, token)
├── data/
│   └── bookings.json                           # Data-driven test inputs
├── reports/                                    # Generated reports (gitignored)
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
| `npm run test:data` | Run the CRUD flow once per row in `data/bookings.json`. |
| `npm run test:html` | Run and export an HTML report to `reports/report.html`. |
| `npm run test:ci` | Data-driven run with HTML + JUnit reports (used by CI). |

Example:

```bash
npm run test:html
open reports/report.html   # macOS
```

## Test coverage

| Folder | Requests |
|---|---|
| **Health Check** | Ping (service up → 201) |
| **Auth** | Get token (valid) · Bad credentials (negative) |
| **Booking - CRUD flow** | Get all · Create · Get by id (verify persisted) · Filter by name · Update (PUT) · Update (PATCH) · Delete · Get deleted (verify gone → 404) |
| **Negative cases** | Unknown booking id (404) · Update without token (403) |

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
2. Sets up Node.js with npm cache.
3. Installs dependencies with `npm ci`.
4. Runs the collection via `npm run test:ci` (CLI + HTML + JUnit reporters).
5. Uploads `reports/` (HTML + JUnit) as a downloadable build artifact, kept for 30 days.
6. Publishes a test-results summary from the JUnit output.

The build **fails if any assertion fails**, so the status badge at the top of this README reflects the real state of the API contract. To download a report, open the latest run under the repo's **Actions** tab and grab the `newman-reports` artifact.

## Notes on Restful Booker

- **Auth:** protected methods (`PUT` / `PATCH` / `DELETE`) require a token from `POST /auth` (`admin` / `password123`), sent as `Cookie: token=...`. The suite handles this automatically.
- **Status codes:** the API returns `201` for both `GET /ping` and a successful `DELETE` — the tests assert these documented behaviours explicitly.
- **Hosting:** Restful Booker runs on a Heroku dyno that may cold-start after inactivity, so the global response-time assertion is intentionally generous and the request timeout is set to 60s.
- **Shared data:** it is a public sandbox; created bookings are visible to everyone and may be periodically reset. The tests are self-contained (they create the data they assert on), so this does not affect results.

## Author

**Samanta Rubio** — QA Automation Engineer
