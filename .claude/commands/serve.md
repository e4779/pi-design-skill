---
description: Start a local http server on 127.0.0.1:4567 to avoid file:// CORS issues. Required for React/Babel artifacts that load external .jsx files.
argument-hint: [port]
allowed-tools: Bash(python3:*) Bash(lsof:*) Bash(sleep:*) Bash(curl:*)
---

# Serve

Start a local http server on port `$0` (default 4567). Needed for any artifact that loads external `.jsx` through `<script src>` + `type="text/babel"` — Babel-standalone uses XHR, which fails on `file://` due to CORS.

## Steps

1. Port: `${0:-4567}`. Check free: `Bash(lsof -i :${port} >/dev/null 2>&1 && echo busy || echo free)`. If busy → try `${port}+1` up to `+5`.
2. Start: `Bash(python3 -m http.server <port> --bind 127.0.0.1)` in background
3. Verify: `Bash(sleep 1 && curl -s -I http://127.0.0.1:<port>/ | head -1)` → expects `HTTP/1.0 200 OK`
4. Report:
   ```
   Server up at http://127.0.0.1:<port>
   Use http://127.0.0.1:<port>/artifacts/<name>.html for /preview or /done.
   To stop: kill the background process via `kill <pid>` or close this session.
   ```

## Notes

- Runs in background — does not block the session
- One server per project is enough; subsequent `/serve` calls just report existing port if already up
- `--bind 127.0.0.1` keeps it loopback-only (not on the network)
