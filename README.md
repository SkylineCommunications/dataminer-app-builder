# DataMiner App Builder

> [!WARNING]
> This is an **experimental** agent used to explore an alternative app-building approach. It may change significantly in the future, and apps built with it today might not work with later versions and may need to be rebuilt.
>
> Distribution of this agent may also change over time and might not remain available through this plugin or repository.

A focused plugin packaging the **DataMiner App Builder** agent and its supporting skills to scaffold, design, and integrate static frontend applications that run inside Skyline DataMiner.

---

## Installation

### VS Code

Open the Command Palette in VS Code (`Ctrl+Shift+P`) and run:

```
Chat: Install Plugin From Source
```

Then enter the repository URL:

```
https://github.com/SkylineCommunications/dataminer-app-builder
```

### Copilot CLI

Install the plugin:

```shell
/plugin install https://github.com/SkylineCommunications/dataminer-app-builder
```

---

## Updating

### VS Code

VS Code checks for plugin updates automatically every 24 hours. To update manually, open the Command Palette in VS Code (`Ctrl+Shift+P`) and run:

```
Extensions: Check for Extension Updates
```

Updates are applied when a new `version` is published in `plugin.json`. After updating, reload the window to activate the latest skills and agent.

### Copilot CLI

```shell
/plugin update dataminer-app-builder
```

---

## Information

### Frontend applications

Apps are static frontend applications (React + Vite + TypeScript) built with AI-assisted development: you describe what you want in natural language and the DataMiner App Builder agent generates the code. The build produces plain HTML, CSS, and JavaScript — no server-side runtime — served by DataMiner at `{PROTOCOL}://{DOMAIN}/public/{BuildFolderName}/index.html`.

They're a good fit when you need a highly customized UI beyond Low-Code Apps, full control over framework and styling, rapid prototyping, or an app that reads, acts on, or writes back DataMiner data (elements, alarms, DOM instances, GQI queries, Automation Scripts).

### Prerequisites

* DataMiner system running version 10.5 or higher
* Node.js

We recommend using Copilot in VS Code or the Copilot CLI. Alternatively, you can manually copy the agent and skills context into your IDE of choice.

### Security

* Never share your password in chat. Enter it only in the terminal when asked.
* Use a dedicated development account that you can easily delete or restrict. Once the agent has access to a system, it can perform almost any action on your behalf, so proceed carefully and make sure you understand what you are doing.

### Deploying to DataMiner

Apps deploy as static files on a DataMiner system. Running `npm run build` produces a folder (`index.html` + `assets/`) that you deploy in one of two ways:

- **Manual** — copy the build folder to `C:\Skyline DataMiner\Webpages\public\<your-app-name>\`. The app is then available at `{PROTOCOL}://{DOMAIN}/public/<your-app-name>/index.html`.
- **CI/CD pipeline** — build and copy the output to the target system as part of your pipeline for consistent, repeatable deployments.

### Expected outlook

These apps are still an early, evolving approach. As adoption grows and the tooling matures, expect improvements across scaffolding, DataMiner integration, deployment, and the App Builder agent itself. The direction is toward more opinionated templates, deeper SDK-level integration, one-click deployment, and standardized best practices.

---

## Repository Structure

```
dataminer-app-builder/
├── plugin.json
├── README.md
├── agents/
│   └── dataminer-app-builder.agent.md
└── skills/
    ├── web-api/
    ├── create-new-app/
    ├── data-discovery/
    ├── execute-automation-script/
    ├── execute-query/
    ├── frontend-design/
    ├── headless-ias/
    ├── performing-actions/
    └── debug-issues/

```

### `agents/`

| Agent | Purpose |
|---|---|
| [`dataminer-app-builder`](./agents/dataminer-app-builder.agent.md) | Builds static frontend applications for deployment inside Skyline DataMiner |

To invoke the agent, use its name in a Copilot prompt or select the agent in the input e.g.:
```
Use the DataMiner App Builder to create a new app that shows active alarms.
```

### `skills/`

| Skill | Purpose |
|---|---|
| [`web-api`](./skills/web-api/SKILL.md) | DataMiner Web Services endpoints: authentication, elements, alarms, services, and WebSocket setup |
| [`create-new-app`](./skills/create-new-app/SKILL.md) | End-to-end guidance for scaffolding a new DataMiner app from scratch |
| [`data-discovery`](./skills/data-discovery/SKILL.md) | Fetching data from DataMiner using GQI, DOM instances, and custom queries |
| [`execute-automation-script`](./skills/execute-automation-script/SKILL.md) | Running a DataMiner Automation Script via `ExecuteAutomationScriptWithOutput` |
| [`execute-query`](./skills/execute-query/SKILL.md) | Executing a known GQI query over WebSocket with paging (`OpenQuerySessionAsync`) |
| [`frontend-design`](./skills/frontend-design/SKILL.md) | Creating and updating the user interface - layout, components, and Skyline design conventions |
| [`headless-ias`](./skills/headless-ias/SKILL.md) | Driving an Interactive Automation Script from a custom UI over WebSocket |
| [`performing-actions`](./skills/performing-actions/SKILL.md) | Write operations on DataMiner: set, create, update, and delete |
| [`debug-issues`](./skills/debug-issues/SKILL.md) | Debugging issues in the application |

---

## Contributing

- **New agent profiles** — add a `<name>.agent.md` file to the `agents/` directory.
- **New skills** — add a subdirectory to `skills/` containing at least a `SKILL.md` file.
- **Profile content** — update this `README.md`.
