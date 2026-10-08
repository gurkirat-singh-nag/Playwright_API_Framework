# Playwright API Framework

A generic, reusable test automation framework for REST APIs, built with [Playwright](https://playwright.dev/docs/api-testing) and TypeScript. It supports multiple test environments (DEV, SIT, UAT) out of the box and keeps test data separate from test logic.

The sample tests run against the public [Restful Booker](https://restful-booker.herokuapp.com) API. To test your own service, point the configuration at your endpoints and add your own specs and test data.

## Features

- **Reusable API client:** one wrapper for `GET`, `POST`, `PUT`, `PATCH` and `DELETE`, with optional token authentication and response logging.
- **Playwright fixtures:** ready-made API clients are injected into every test.
- **Multi-environment configuration:** switch between DEV, SIT and UAT with a single environment variable.
- **Externalized test data:** request bodies are stored as JSON files for each environment.
- **HTML reporting:** Playwright's built-in list and HTML reporters.
- **CI-friendly:** retries, worker count and `forbidOnly` adjust automatically when `CI` is set.

## Tech Stack

| Tool | Purpose |
| --- | --- |
| [Playwright Test](https://playwright.dev) | Test runner and HTTP request context |
| TypeScript | Language |
| Node.js | Runtime |

## Project Structure

```
.
├── playwright.config.ts              # Global and environment-specific configuration
├── package.json                      # Scripts and dependencies
├── tsconfig.json
└── src/test
    ├── constants/
    │   └── api-request-constant.ts   # Loads JSON test data for the active environment
    ├── fixtures/
    │   └── api-fixture.ts            # Custom Playwright fixtures that provide API clients
    ├── utils/
    │   └── api-client.ts             # Generic HTTP client wrapper
    ├── resources/                    # Test data for each environment
    │   ├── dev/
    │   ├── sit/
    │   └── uat/
    └── specs/                        # Test suites
        ├── auth.spec.ts
        └── booking.spec.ts
```

## Prerequisites

- Node.js 18 or later
- npm

## Installation

```bash
npm ci
```

## Running Tests

| Environment | Command |
| --- | --- |
| DEV | `npm run test-dev` |
| SIT | `npm run test-sit` |
| UAT | `npm run test-uat` |

Each command runs the full suite and then opens the HTML report.

> **Windows note:** the npm scripts use the Unix syntax `TEST_ENV=dev ...`. In PowerShell, set the variable first and then run Playwright:
>
> ```powershell
> $env:TEST_ENV = "sit"; npx playwright test
> ```

Other useful commands:

```bash
npx playwright test src/test/specs/auth.spec.ts   # run a single spec
npx playwright test -g "Booking"                  # run tests whose title matches
npx playwright show-report                        # open the last HTML report
```

If `TEST_ENV` is not set, the framework uses **dev**.

## Configuration

Environment settings are defined in [playwright.config.ts](./playwright.config.ts). Each environment provides:

| Property | Description |
| --- | --- |
| `authApiUrl` | Endpoint used to obtain an authentication token |
| `baseApiUrl` | Base URL for the API under test |
| `testDataDir` | Folder containing that environment's JSON test data |

### Adding a new environment

1. Create a config object in `playwright.config.ts`, following the pattern of `devConfig`, `sitConfig` and `uatConfig`.
2. Add it to the environment selection logic in the same file.
3. Create a matching folder under `src/test/resources/` with the required JSON files.
4. Optionally add an npm script such as `"test-prod": "TEST_ENV=prod npx playwright test"`.

## Writing Tests

### 1. Add test data

Create a JSON request body in each environment folder, for example `src/test/resources/dev/create-user-request.json`.

### 2. Register it as a constant

```ts
// src/test/constants/api-request-constant.ts
export const CREATE_USER_REQUEST_JSON_BODY = loadJsonFile("create-user-request.json");
```

### 3. Write the spec

```ts
import { expect } from "@playwright/test";
import { fixtures as test } from "../fixtures/api-fixture";
import { CREATE_USER_REQUEST_JSON_BODY } from "../constants/api-request-constant";

test.describe("User Test Suite", () => {
  test("[POST] Create a user", async ({ commonApiFixture }) => {
    const response = await commonApiFixture.post("/users", CREATE_USER_REQUEST_JSON_BODY);
    expect(response.status()).toBe(201);
  });
});
```

### Available fixtures

| Fixture | Base URL |
| --- | --- |
| `authApiFixture` | `authApiUrl` |
| `commonApiFixture` | `baseApiUrl` |

### API client methods

```ts
get(endpoint, token?)
post(endpoint, body?, token?)
put(endpoint, body?, token?)
patch(endpoint, body?, token?)
delete(endpoint, token?)
```

When you pass a token, the client sends it as a `Cookie: token=<value>` header. If your API expects a different scheme, such as `Authorization: Bearer <token>`, change the header in [api-client.ts](./src/test/utils/api-client.ts).

## Reports

After a run, the HTML report is saved to `playwright-report/` and other test artifacts to `test-results/`. Neither folder is committed to git.

## Author

**Gurkirat Singh**
