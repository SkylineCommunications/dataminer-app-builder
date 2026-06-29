---
name: debug-issues
description: Use when the user says something is broken, not working, has a bug, or behaves unexpectedly. Open the running app in a browser and use Playwright to inspect console logs, network requests, and WebSocket frames to find the cause yourself before asking the user.
user-invocable: true
---

# Debug in the browser

When the user reports an issue, reproduce and diagnose it yourself before asking questions.

## Steps

1. Make sure the dev server is running (`npm run dev`). Start it if needed, and note the URL it prints.
2. Open the page with `open_browser_page`.
3. If the app redirects to the DataMiner sign-in page → **See [Authenticate](#authenticate) below before going any further.**
4. Use `run_playwright_code` to attach listeners, then trigger the broken flow.
5. Read what came back, find the mistake, fix it, re-check.

---

## Authenticate

Do this immediately after opening the page — before reading source code, before any analysis.

Check the current URL of the shared browser page:

- If it contains `/auth/login` or any other sign-in path → the app has redirected to the DataMiner login page.
- Send this message to the user:

  > "The app redirected to the DataMiner sign-in page. Please sign in at `<URL>` in the shared browser, then let me know once you're past the login and I'll continue."

Rules:

- Wait for the user to confirm they're signed in before doing anything else.
- Never ask for credentials — the user signs in themselves in the browser.
- Never skip this step

---

## Playwright snippet to capture everything

```js
const logs = [];
page.on('console', m => logs.push(`[console.${m.type()}] ${m.text()}`));
page.on('pageerror', e => logs.push(`[pageerror] ${e.message}`));
page.on('requestfailed', r => logs.push(`[reqfail] ${r.url()} ${r.failure()?.errorText}`));
page.on('response', r => { if (!r.ok()) logs.push(`[http ${r.status()}] ${r.url()}`); });
page.on('websocket', ws => {
  logs.push(`[ws open] ${ws.url()}`);
  ws.on('framesent', f => logs.push(`[ws >>] ${f.payload}`));
  ws.on('framereceived', f => logs.push(`[ws <<] ${f.payload}`));
});

await page.goto('<APP_URL>'); // e.g. http://localhost:5173 — confirm the real host/port first
// trigger the broken action here, e.g. await page.click('text=Facilities');
await page.waitForTimeout(2000);
return logs.join('\n');
```

---

## Pick what to listen for

- **Blank screen / crash** → `console` + `pageerror`.
- **Data not loading** → `response` (non-2xx) + `requestfailed`.
- **GQI / live data wrong** → `websocket` frames (sent query + received rows).
- **Wrong UI state** → `read_page` / `screenshot_page` to inspect the DOM.

---

## Rules

Inspect first, conclude from real evidence, then fix. Only ask the user if the cause genuinely can't be observed (e.g. needs their credentials or a manual action).