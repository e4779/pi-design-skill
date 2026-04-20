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
2. Open in default browser: `Bash(open "<url>")` (macOS) or `Bash(xdg-open "<url>")` (linux)
3. Navigate Chrome DevTools MCP: `mcp__chrome-devtools__navigate_page({url: "<url>"})`
4. Report: preview is live; screenshots and console inspection now work via the MCP session
