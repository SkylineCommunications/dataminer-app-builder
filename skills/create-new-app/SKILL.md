---
name: create-new-app
description: 'This skill helps you create a new frontend application with DataMiner API integration from zero. It guides you through gathering requirements, setting up the project, implementing initial features, and ensuring best practices for the base structure of your application.'
user-invocable: true
---

# Create New App Skill

Before implementing any new app, read and follow the [dataminer-e2e-testing skill](../dataminer-e2e-testing/SKILL.md). Every created app MUST include passing frontend-only Playwright tests, even when tests were not explicitly requested. Mock all DataMiner HTTP APIs and WebSockets; the suite must not require a live DMA, real credentials, or backend operations.

When creating a new application you should ALWAYS go through the following steps:

1. Gather requirements from the user
2. Discover and verify the required DataMiner data
3. Set up the project structure and build configuration
4. Handle session bootstrap (no in-app login UI)
5. Implement initial features against the discovered contracts
6. Build the application and run its mandatory mocked frontend Playwright suite
7. Provide deployment instructions for hosting on DataMiner

Make sure you don't skip any of these steps and follow them in order to ensure a successful app creation process.

---

1. Gather requirements from the user

- Ask the user what they want in their frontend app
- Never skip this step and start building immediately
- Get specific requirements, features, data and functionality details
- Understand the user's exact needs before proceeding

---

2. Discover and verify the required DataMiner data

Data discovery is a required gate before designing the data layer, writing production data-access code, or creating test fixtures.

1. Turn every requested screen, metric, filter, and action into explicit data questions.
2. Use the `web-api` skill to identify documented endpoints for elements, alarms, views, and services. Use the `data-discovery` skill for DOM instances, ad hoc data sources, and custom GQI data.
3. For GQI data, run the complete agent-side discovery flow against the user's DataMiner system. Capture the generated query object, execute it once using the `execute-query` skill, and inspect a representative result page so column names, types, nullability, and identifiers are known.
4. Record a concise data contract in the app source: the exact endpoint or hardcoded GQI query, the response-to-view-model mapping, and representative sanitized rows. Never invent an aggregate endpoint, field, or response shape merely to simplify the frontend.
5. Confirm that the discovered data supports every required UI behavior. If data is unavailable or ambiguous, resolve it with the user before implementation rather than substituting placeholder production data.

Discovery uses the live system only during app creation. Do not ship credentials, discovery scripts, raw sensitive records, or `CreateAIGeneratedQuery` calls in the application. Sanitize representative values before using them as test fixtures.

---

3. Set up the project structure and build configuration

- **Framework:** React
- **Output:** Static build (HTML / CSS / JS)
- Do not use `npm create vite` or any CLI scaffold. Create all files directly, then run `npm install` once.
- All app files must be created inside a dedicated project subfolder (e.g. `my-app-name/`) — never at the workspace root.
- Always use the latest version of dependencies (React, Vite, TypeScript, etc.) unless the user specifies otherwise or a specific version is required for compatibility.
- Include `@playwright/test` as a dev dependency, a `test:e2e` script, `playwright.config.ts`, and an `e2e/` directory as required by `dataminer-e2e-testing`.

**Architecture Constraints**

- No backend, proxies, or server middleware
- Lightweight and optimized
- All logic runs client-side

**Deployment model:**

- The build will be placed directly onto the hosting platform
- No server-side runtime required
- No Node runtime required after build

**Build configuration requirements:**

- A dedicated production build path
- A clean output folder
- No unnecessary dependencies
- Optimized, minimal bundle size
- Production build output is served from `/public/{BuildFolderName}/` on the DataMiner host
- All API and auth URLs must be relative — no hardcoded hosts or protocols
- Set `outDir` and `base` to the project name. For example, for a project named `my-app`: `outDir: 'my-app'` and `base: '/public/my-app/'`.

**Dependency management guidelines:**

- Avoid third-party npm packages unless they provide significant value
- Do not reinvent the wheel - use established packages for complex functionality (e.g., date manipulation, charting)
- Prefer native browser APIs and vanilla JavaScript when feasible

---

4. Handle session bootstrap (no in-app login UI)

- Do NOT build an in-app login page. Users sign in through DataMiner's built-in `/auth/` page before the app loads.
- Users reach the app via `{PROTOCOL}://{DOMAIN}/auth/?url=%2Fpublic%2F{BuildFolderName}%2Findex.html`.
- The app must expose a sign-out action.

For all session implementation details (cookie bootstrap, GUID verification, auth guard, sign-out flow), follow the **web-api** skill (rules 3, 4, 5).

---

5. Implement initial features based on requirements

- Do not add unasked features or assumptions beyond the requirements provided by the user

### Fetching data from DataMiner

Use the exact documented endpoints or GQI queries established during data discovery. Never use direct DOM/element SOAP calls, and never replace the discovered production calls with a made-up frontend-specific API.

Use the **data-discovery** skill to discover GQI queries and the **execute-query** skill to execute them in the production app. The two-step approach is:

1. **Discover**: The agent authenticates to the DataMiner system after collecting the host and username; the user enters the password directly into the terminal. The agent then runs an agent-side Node.js script that calls `CreateAIGeneratedQuery` (NL2GQI) and captures the discovered query object.
2. **Build**: The agent hardcodes the discovered query into the production app using `OpenQuerySessionAsync`. The app itself never calls `CreateAIGeneratedQuery`.

---

6. Build and test the application to ensure it meets requirements and is production-ready

- Before creating or extending tests, write `e2e/TEST_PLAN.md` with app-specific scenarios, expected UI outcomes, and explicit exclusions. Use Chromium only; authentication is mocked fixture setup, not a test target.
- Run `npm run build` and require a successful production build
- Create and run the mandatory `e2e/` suite with `npx playwright test`, served from the production build under `/public/{BuildFolderName}/`
- Store each run under `output/tests/` with machine-readable results at `results.json`, an HTML report at `report/index.html`, and failure evidence under `artifacts/`, following `dataminer-e2e-testing`
- Cover every requested app feature, combined interactions, resets/recovery, empty/edge states, supported data-error states, responsive usability, and rendered theming when present, following `dataminer-e2e-testing`
- Mock every external HTTP and WebSocket dependency; test the real frontend without calling a live DataMiner system or testing backend behavior
- Mock the exact production endpoints, request bodies, WebSocket methods, subscription flow, and response envelopes identified during discovery. Build happy-path fixture rows from sanitized discovery output, then derive empty, edge, update, and error fixtures from that same schema.
- Keep mocks exclusively in the Playwright layer. Production components and data clients must not import test fixtures, switch to mock data based on test mode, or call a development-only aggregate endpoint.
- Verify the output is functional and deployable
- Do not mark the app complete until the suite exists and every required test passes; missing, failing, skipped required, or unrun tests block completion
- If build or test execution is blocked, report the blocker explicitly instead of claiming completion

---

7. Provide deployment instructions for hosting on DataMiner

Inspect `output/tests/results.json`, then include the build/test commands, actual passing test counts, and links to `e2e/TEST_PLAN.md` and `output/tests/report/index.html` in the handover. Keep the latest test output available after validation. Deployment instructions do not replace the mandatory test gate.

After giving the user a new build, say:
> "We added a build folder to the project named `BuildFolderName`. You should copy that folder and paste it in the \Skyline DataMiner\Webpages\Public folder. Then access it through DataMiner's sign-in page at: `http(s)://<your-dma>/auth/?url=%2Fpublic%2F{BuildFolderName}%2Findex.html` (the `url` value is URL-encoded — `%2F` represents `/`). Opening `/public/{BuildFolderName}/index.html` directly will not have a session."
