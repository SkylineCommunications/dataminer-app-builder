---
name: performing-actions
description: 'Use when the app needs to perform an action on the DataMiner system: creating, updating, deleting data, triggering workflows, or executing any server-side operation. Covers which mechanism to use and how to call it from the frontend.'
---

# Performing Actions Skill

## When to Use
- The user wants to create, update, or delete data in DataMiner
- The app needs to trigger a server-side operation
- Any frontend interaction that writes data back to DataMiner

## Available Mechanisms

### Execute Automation Script
Run a DataMiner Automation Script to perform an operation server-side.

Use this when there is an existing Automation Script that handles the operation.

See the [execute-automation-script](../execute-automation-script/SKILL.md) skill for the full API details, request structure, and parameter encoding rules.

### User-Defined APIs (UDAPIs)
A UDAPI works only once its logic lives in an Automation Script — the user must define the API on top of a script (see [Using an existing script as a User-Defined API](https://docs.dataminer.services/dataminer/Functions/User-Defined_APIs/Defining_an_API/UD_APIs_Using_existing_scripts.html)).

Do not call the [UDAPI](https://docs.dataminer.services/dataminer/Functions/User-Defined_APIs/UD_APIs.html) HTTP endpoint directly from the app — UDAPIs need a bearer token, and embedding one in client-side code exposes it. Instead, run that script's `Run` entry point via `ExecuteAutomationScriptWithOutput` (see above), authenticated by the DataMiner session — no bearer token needed. The script returns data with `engine.AddScriptOutput(key, value)`, which comes back as the `{ Key, Value }` output list.

## General Pattern

1. Identify the right mechanism and operation to call
2. Call it using the appropriate API function. Same-origin requests automatically carry the `DMAConnection` session cookie; pass the in-memory `connection` GUID only where the endpoint explicitly requires it (see the **web-api** skill)
3. Pass the required parameters
4. On success: trigger a data refresh so the UI reflects the change
5. On error: surface the error message to the user in the UI
