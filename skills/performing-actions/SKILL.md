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

## General Pattern

1. Identify the right mechanism and operation to call
2. Call it using the appropriate API function. Same-origin requests automatically carry the `DMAConnection` session cookie; pass the in-memory `connection` GUID only where the endpoint explicitly requires it (see the **web-api** skill)
3. Pass the required parameters
4. On success: trigger a data refresh so the UI reflects the change
5. On error: surface the error message to the user in the UI
