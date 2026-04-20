---
description: Open HTML in browser and navigate Chrome DevTools MCP to same URL
argument-hint: <html-path>
allowed-tools: Bash(open:*) Bash(xdg-open:*) Bash(realpath:*) Bash(readlink:*) mcp__chrome-devtools__navigate_page
---

Preview the HTML file or URL at `$0` in the user's browser and navigate the Chrome DevTools MCP session to the same URL.

## Argument forms

- Path (e.g. `artifacts/deck.html`) → converted to `file://<abs>`
- `http://...` / `https://...` → used as-is (use this for artifacts that need `/serve` due to React+Babel CORS)

## Steps

1. If `$0` starts with `http://` / `https://`, use as `url`. Otherwise resolve to absolute path and prepend `file://`:
   ```
   !`realpath "$0" 2>/dev/null || readlink -f "$0" 2>/dev/null || (cd "$(dirname "$0")" && pwd)/$(basename "$0")`
   ```
2. **Cache-bust** (important): append a `?v=<epoch-ms>` query so the browser doesn't serve a stale cached render after an `Edit` to the HTML. Generate via `Bash(date +%s%3N 2>/dev/null || date +%s000)`. Resulting URL: `file://<abs>?v=1729440000000`.
   - Browsers ignore unknown query params on `file://`, so this is safe.
   - For `http(s)://` URLs, same pattern works unless the server strips query strings.
3. Open in default browser: `Bash(open "<url>")` (macOS) or `Bash(xdg-open "<url>")` (linux)
4. Navigate Chrome DevTools MCP: `mcp__chrome-devtools__navigate_page({url: "<url>", type: "url"})` — the fresh `?v=` query forces a network fetch even if the tab was previously loaded.
5. Report: preview is live; screenshots and console inspection now work via the MCP session
