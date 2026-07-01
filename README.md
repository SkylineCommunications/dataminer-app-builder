# DataMiner App Builder

A focused plugin containing the **DataMiner App Builder** agent and its supporting skills for building static frontend applications that run inside Skyline DataMiner.

## What is this?

This plugin packages a single agent and the skills it needs to scaffold, design, and integrate DataMiner frontend apps — from a blank workspace all the way to a deployed application.

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
    └── performing-actions/
    └── debug-issues/

```

### `agents/`

| Agent | Purpose |
|---|---|
| [`dataminer-app-builder`](./agents/dataminer-app-builder.agent.md) | Builds static frontend applications for deployment inside Skyline DataMiner |

To invoke the agent, use its name in a Copilot prompt, e.g.:
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
