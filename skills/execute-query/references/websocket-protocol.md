# WebSocket Protocol — GQI Reference

For generic WebSocket connection setup (URL, `SetConnectionID`, keep-alive, message structure, `ClientSubscriptionID` management), see the **web-api** skill (`references/websocket-setup.md`).

This reference covers GQI-specific WebSocket messages used by `CreateAIGeneratedQuery` and `OpenQuerySessionAsync`.

---

## Queue-based event subscription

HTTP calls that trigger async work (e.g. `CreateAIGeneratedQuery`, `OpenQuerySessionAsync`) create a server-side queue. Subscribe to it via WebSocket **after** the HTTP call returns:

```json
{ "ClientSubscriptionID": <N>, "Method": "GetEvents", "Parameters": { "queueID": <same-queueID-as-HTTP-call> } }
```

The server then delivers results as `DMAEvent` messages on that queue.

> **Ordering rule:** Register the WebSocket `message` listener BEFORE the HTTP call. Send `GetEvents` AFTER the HTTP call returns (the queue doesn't exist server-side until the HTTP call creates it).

---

## Requesting GQI row pages

Once you have a session ID from `OpenQuerySessionAsync`, request pages via WebSocket (not HTTP):

```json
{
  "ClientSubscriptionID": <N>,
  "Method": "NotifySubscription",
  "Parameters": {
    "clientSubscriptionID": <N>,
    "method": "GetNextQuerySessionPage",
    "data": { "sessionID": "<session-id>", "pageSize": 50 }
  }
}
```

The server responds with a `DMAEvent`:

```json
{
  "Type": "DMAEvent",
  "Data": {
    "Message": {
      "Rows": [
        { "Cells": [{ "Value": "raw-value", "DisplayValue": "display-string" }] }
      ],
      "IsLast": false
    }
  }
}
```

- Use `DisplayValue` for rendering — it resolves discreet values to human-readable strings
- `IsLast: false` → send another `GetNextQuerySessionPage`
- `IsLast: true` → all rows received

---
