---
name: doctor
description: First-run setup + health check. Verifies Chrome DevTools MCP, installs optional deps (monolith, pptxgenjs, puppeteer), creates working dirs, runs smoke test. Use on fresh clone or when things feel broken.
allowed-tools: read write mcporter
---

# Doctor

Cold-start health check + optional auto-install. Safe to run multiple times.

## Phase 1 — Inventory

Run in parallel where possible:

1. `` — parse output, look for `chrome-devtools: ✓ Connected`
2. `` — existence check
3. `` — required
4. `` — for `/ingest-github`
5. ``
6. `` — expect yes
7. ``
8. `` — expect yes

## Phase 2 — Report

Print structured report:

```
Chrome DevTools MCP: ✓ / ✗ (if ✗, show install command)
monolith CLI: ✓ / ✗ (optional; needed for /export-standalone)
Node: ✓ / ✗ (required)
gh CLI: ✓ / ✗ (needed for /ingest-github)
package.json: exists / missing
starters/ dir: exists / missing
artifacts/ dir: will create
.claude/skills/: N skills found
FIGMA_TOKEN env: set / not set (only needed for /ingest-figma)
```

## Phase 3 — Repair (with user consent)

For each missing item, ask the user with `AskUserQuestion`: "Install X now?" (yes/skip/all).

Repair commands (only on yes):
-*Chrome DevTools MCP:** `` — note: requires Claude Code restart to take effect
-*monolith:** `` (macOS) or tell user for linux
-*pptxgenjs + puppeteer:**
 ```


 ```
-*artifacts/ dir:** ``

## Phase 4 — Smoke test

If everything is green (and user didn't just run install → restart):
1. write `test/smoke-deck.html`:
 ```html
 <!doctype html>
 <html><head><meta charset="utf-8"/><title>Smoke</title>
 <script src="../starters/deck_stage.js"></script></head>
 <body><deck-stage width="1920" height="1080">
 <section style="padding:60px;font:48px/1.2 Georgia"><h1>One</h1></section>
 <section style="padding:60px;font:48px/1.2 Georgia;background:#f4e4d7"><h1>Two</h1></section>
 <section style="padding:60px;font:48px/1.2 Georgia;background:#4a5d2e;color:#fff"><h1>Three</h1></section>
 </deck-stage></body></html>
 ```
2. Run `/done test/smoke-deck.html`
3. If Chrome DevTools MCP is connected — check screenshot at `.claude/last-preview.png` exists and console is clean
4. Report pass/fail

## Phase 5 — Hint sheet

Print one-liner hints:
```
Available commands & skills:
 /make-deck <brief> — build a slide deck
 /interactive-prototype <brief> — clickable React prototype
 /wireframe <thing> — low-fi variants
 /create-design-system <src> — extract style guide
 /ingest-github <url> — pull tokens from repo
 /ingest-screenshot <path> — extract tokens from image
 /ingest-figma <url> — pull from Figma (needs FIGMA_TOKEN)
 /make-tweakable <html> — add live tweak panel
 /apply-tweaks <html> — persist panel changes
 /register-asset <html> — add to assets.html overview
 /verify-artifact <html> — vision-based QA
 /export-pdf /export-pptx /export-standalone /handoff
 /preview /done /screenshot — operational atomics
```
