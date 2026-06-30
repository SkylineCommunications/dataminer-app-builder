---
name: execute-query
description: 'Execute GQI queries in a DataMiner frontend application using OpenQuerySessionAsync over WebSocket. Use when building production apps that need to fetch GQI data (DOM instances, ad hoc data sources, custom queries) from DataMiner using a known query object, or when the user provides a query object directly.'
user-invocable: true
---

# Execute Query Skill

GQI is for data that is **NOT** available through the standard DataMiner Web API endpoints. Use GQI for DOM instances, ad hoc data sources, and custom queries. Do **NOT** use GQI for elements, alarms, views, or services — those are available through the **web-api** skill.

This skill covers executing a **known** GQI query in a production app. A query object can come from two sources:
- **Discovered** via natural language using the **data-discovery** skill (NL2GQI)
- **Provided directly** by the user as a query object

Official references:
- GQI: https://docs.dataminer.services/dataminer/Functions/Dashboards_and_Low_Code_Apps/GQI/About_GQI.html

For **real-time (push) updates** — subscribing to live row changes via `ObserveQuerySessionAsync` instead of repeated one-shot fetches — see `references/realtime-updates.md`.

---

## Overview

GQI query execution happens in two phases:

1. **Open a session** — HTTP POST to `OpenQuerySessionAsync` with the hardcoded query object. Returns a session ID and column metadata.
2. **Fetch pages** — Request row data pages via WebSocket using `GetNextQuerySessionPage` until `IsLast: true`.

Both phases share a single WebSocket connection. For WebSocket connection setup (URL, `SetConnectionID`, keep-alive, `ClientSubscriptionID` management), see the **web-api** skill (`references/websocket-setup.md`). The `connection` string comes from the session bootstrap call described in the **web-api** skill.

See `references/websocket-protocol.md` for GQI-specific message formats (queue events and row data pages).

---

## OpenQuerySessionAsync HTTP call

```json
POST ../../API/v1/Internal.asmx/OpenQuerySessionAsync
{
  "queueID": <number>,
  "clientSubscriptionID": <number>,
  "connection": "<connection-string>",
  "query": { <hardcoded query object> },
  "options": {
    "OptimizationType": 1,
    "TimeZoneID": "Europe/Brussels",
    "LanguageTag": "en-US",
    "FetchLocal": true,
    "UseDynamicUnits": true,
    "EnableUpdates": false,
    "QueryTag": ""
  }
}
```

**`options` field reference:**

| Field | Type | Description |
|-------|------|-------------|
| `OptimizationType` | int | `1` = standard. Controls server-side query optimization strategy. |
| `TimeZoneID` | string | IANA timezone string used to format date/time values (e.g. `"Europe/Brussels"`). Use `Intl.DateTimeFormat().resolvedOptions().timeZone` for the user's local timezone. |
| `LanguageTag` | string | BCP 47 language tag for localized display values (e.g. `"en-US"`). Use `navigator.language` for the user's browser language. |
| `FetchLocal` | bool | `true` = data is fetched on the DMA itself (required for most use cases). |
| `UseDynamicUnits` | bool | `true` = units adapt to the magnitude of the value (e.g. KB → MB → GB). |
| `EnableUpdates` | bool | `false` = one-shot fetch. `true` = live updates pushed via WebSocket as data changes. Use `false` for most production apps. |
| `QueryTag` | string | Optional label for server-side logging and diagnostics. Leave empty unless debugging. |

---

## Response formats

### OpenQuerySessionAsync response

```json
{
  "__type": "Skyline.DataMiner.Web.Common.v1.DMAGenericInterfaceSessionInfo",
  "ID": "<session-guid>",
  "Columns": [
    { "Name": "Column Name", "ClientType": "string", "ServerType": "string", "ID": "column-id", "Discreets": [], "IsDiscreet": false }
  ],
  "ColumnLinks": [ ... ]
}
```

- `d.ID` = session ID (needed for `GetNextQuerySessionPage`)
- `d.Columns` = column metadata (names, types, discreet values)

### Row data format

```json
{
  "Type": "DMAEvent",
  "Data": {
    "Message": {
      "Rows": [
        { "Cells": [{ "Value": "actual-value", "DisplayValue": "display-string" }] }
      ],
      "IsLast": true
    }
  }
}
```

- Use `DisplayValue` for rendering — it resolves discreet values to their display strings
- Enum-like columns return their display string, not an integer — key filters/lookups on the string.
- `IsLast: false` means more pages are available — send another `GetNextQuerySessionPage`

---

## WebSocket execution sequence

| Step | Direction | Action |
|------|-----------|--------|
| 1 | client→server | `SetConnectionID` — bind WebSocket to HTTP session |
| 2 | server→client | `Type: "String"` confirmation — wait before proceeding |
| 3 | client→HTTP | `OpenQuerySessionAsync` — creates queue; returns `d.ID` (session ID) and `d.Columns` |
| 4 | client→server | `GetEvents` with same `queueID` — subscribe to queue |
| 5 | client→server | `GetNextQuerySessionPage` — request first page of rows |
| 6 | server→client | `DMAEvent` with `Message.Rows` — accumulate; check `Message.IsLast` |
| 7 | (repeat 5–6) | if `IsLast: false`, send another `GetNextQuerySessionPage` |

> **Race condition:** Register the WebSocket `message` listener BEFORE calling `OpenQuerySessionAsync`. Send `GetEvents` AFTER the HTTP call returns (the server-side queue doesn't exist until the HTTP call creates it).

---

## Production code example

```javascript
const INTERNAL_API = `${window.location.protocol}//${window.location.host}/API/v1/Internal.asmx`;

async function internalPost(method, body) {
  const r = await fetch(`${INTERNAL_API}/${method}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body)
  });
  const json = await r.json();
  if (!r.ok) throw new Error(json?.Message || r.statusText);
  return json.d;
}

// Hardcoded query object — discovered by the agent's Node.js script (see data-discovery skill)
const QUERY = { /* discovered query object goes here */ };

async function fetchData(connection) {
  let subID = 1;
  const rowQueueID = Math.floor(Math.random() * 1_000_000);

  return new Promise((resolve, reject) => {
    const proto = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
    const ws = new WebSocket(`${proto}//${window.location.host}/API/v1/WebSocket.ashx`);

    let sessionId = null;
    let columns = [];
    const rows = [];

    ws.addEventListener('open', () => {
      ws.send(JSON.stringify({ ClientSubscriptionID: subID++, Method: 'SetConnectionID', Parameters: { connectionID: connection } }));
    });

    ws.addEventListener('message', async e => {
      const msg = JSON.parse(e.data);
      if (msg.Type === 'String') {
        const sessionInfo = await internalPost('OpenQuerySessionAsync', {
          connection,
          queueID: rowQueueID,
          clientSubscriptionID: subID++,
          query: QUERY,
          options: {
            OptimizationType: 1,
            TimeZoneID: Intl.DateTimeFormat().resolvedOptions().timeZone,
            LanguageTag: navigator.language,
            FetchLocal: true,
            UseDynamicUnits: true,
            EnableUpdates: false,
            QueryTag: ''
          }
        });
        sessionId = sessionInfo.ID;
        columns = sessionInfo.Columns;
        ws.send(JSON.stringify({ ClientSubscriptionID: subID++, Method: 'GetEvents', Parameters: { queueID: rowQueueID } }));
        ws.send(JSON.stringify({
          ClientSubscriptionID: subID++,
          Method: 'NotifySubscription',
          Parameters: { clientSubscriptionID: subID++, method: 'GetNextQuerySessionPage', data: { sessionID: sessionId, pageSize: 50 } }
        }));
      }
      if (msg.Type === 'DMAEvent' && msg.Data?.Message?.Rows && sessionId) {
        rows.push(...msg.Data.Message.Rows);
        if (msg.Data.Message.IsLast) {
          ws.close();
          resolve(rows.map(row =>
            Object.fromEntries(columns.map((col, i) => [col.Name, row.Cells[i]?.DisplayValue]))
          ));
        } else {
          ws.send(JSON.stringify({
            ClientSubscriptionID: subID++,
            Method: 'NotifySubscription',
            Parameters: { clientSubscriptionID: subID++, method: 'GetNextQuerySessionPage', data: { sessionID: sessionId, pageSize: 50 } }
          }));
        }
      }
    });

    ws.addEventListener('error', reject);
  });
}
```