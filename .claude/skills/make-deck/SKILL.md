---
name: make-deck
description: Build an HTML slide deck (1920×1080, keyboard nav, exportable) when user asks for a presentation, pitch deck, slides, or keynote. Uses deck_stage.js starter and Claude Design taste rules.
argument-hint: <brief or topic>
allowed-tools: Read Write Edit Glob Grep Bash(cp:*) Bash(open:*) Bash(mkdir:*) Bash(realpath:*) mcp__chrome-devtools__*
---

# Make a Deck

Build an HTML slide deck that looks designed, not AI-generated. Uses `starters/deck_stage.js` for keyboard nav, scaling, speaker notes, and print-to-PDF.

## Phase 0 — Context pre-flight (auto-detect, ONE question max)

Before asking design questions, detect context **silently** via auto-checks. Only ask the user if nothing is found.

### Auto-detect (no user input)
1. `Read .claude/design-tokens.json` — project tokens, if exist → load
2. `Bash(ls ~/.claude/design-systems/ 2>/dev/null)` — org-level registry. If the brief mentions a brand name matching a folder → auto-apply it (e.g. brief "Acme deck" + `~/.claude/design-systems/acme/` → use Acme)
3. `Glob **/tailwind.config.* **/theme.{ts,js,json} **/tokens.{css,scss} **/_variables.*` at project root — if found, note "codebase detected" and `Read` them
4. Scan user message/attachments for:
   - GitHub URL → invoke `Skill: ingest-github` first
   - Figma URL → invoke `Skill: ingest-figma` first
   - Image attachment → invoke `Skill: ingest-screenshot` first
   - `.md` / `.txt` / `.pdf` file attached → `Read` it (may contain brand refs or content)

### One-question fallback (only if nothing detected)
If no context found after auto-detect, use `AskUserQuestion` with **a single question**, text-options:
- **Use existing design system** — list names from `~/.claude/design-systems/` (if any)
- **From codebase** — "paste a local path or github URL"
- **From screenshot** — "attach an image"
- **From Figma** — "paste a figma URL (needs FIGMA_TOKEN)"
- **No context** — invoke `Skill: frontend-design` for aesthetic-from-scratch
- **Decide for me** — Claude picks frontend-design

Report what was found/chosen in one line: "Using <context>. Proceeding…"

## Phase 1 — Ambiguity gate

Check the user's brief (`$ARGUMENTS`) for these signals:
- **Audience** — "for X", "for investors", "team", "stakeholders", etc.
- **Style** — named aesthetic ("New Yorker", "WSJ", "minimal", "bold", "Apple-like") or brand reference
- **Length** — "N slides", "short", "quick"

If **≥2 of 3 present** → skip long questionnaire, proceed with max 1-2 clarifying questions.

If **<2 present** → invoke `AskUserQuestion` with:
- Audience (who's watching, emotional state, technical level)
- Tone (formal/casual, funny/serious)
- Length (slide count)
- Visuals (minimal/rich, photos/illustrations/charts)
- Variations (do they want 2-3 deck options or just one)

Ask at least 4 relevant questions even in gate-passed mode if anything is still unclear. **Do not** ask about brand context — Phase 0 already handled it.

## Phase 1.5 — Speaker-notes heuristic (agent decides, no toggle)

Instead of asking user "use speaker notes?", decide from signals:

| Signal in brief | → speaker notes |
|---|---|
| "quick", "standup", "lightning", "recap", <100 chars | **off** — self-sufficient slides, no notes |
| "for investors", "Board meeting", "keynote", "All Hands" | **on** — sparse slides, notes carry narrative |
| Attached PRD / long doc (>500 words) | **on** — slides distill, notes script |
| Only 1-3 slides requested | **off** |
| 10+ slides requested | **on** (probably a story arc) |
| Ambiguous / none of above | ask a single yes/no question |

If on: generate `<script type="application/json" id="speaker-notes">[...]</script>` at the end of `<body>` with one string per slide — full conversational narrative, not bullet-point summary. Keep slides visual-heavy.

If off: skip notes entirely; slides must stand alone.

## Phase 2 — Visual system

Using context gathered in Phase 0, commit to a visual system up front:
- Section-header layout, title layout, image layout
- Max 1-2 background colors across the deck
- Font hierarchy (display / H1 / body / caption)
- Palette (pull from loaded tokens, or generated via `Skill: frontend-design`)

Vocalize the system in one paragraph before writing slides.

## Phase 3 — Build

1. Create `artifacts/<slug>.html` with shell:
   ```html
   <!doctype html>
   <html lang="en">
   <head>
     <meta charset="utf-8"/>
     <title>{{deck title}}</title>
     <script src="./deck_stage.js"></script>
     <style>/* your tokens + slide styles */</style>
   </head>
   <body>
     <deck-stage width="1920" height="1080">
       <section><!-- slide 1 --></section>
       ...
     </deck-stage>
   </body>
   </html>
   ```
2. `Bash(cp starters/deck_stage.js "$(dirname <html>)/")` — copy starter into the **same dir as the HTML** (important for nested paths like `artifacts/foo/deck.html`)
3. Write all slides in one pass. Rules:
   - NO title-only slide as slide 1 (jump into content)
   - Text ≥ 24px on 1920×1080
   - `text-wrap: pretty`
   - Use CSS Grid for layouts
   - No gradients, no emoji, no Inter/Roboto/Arial — pick character fonts
   - No SVG-drawn imagery — placeholder divs with labels instead
   - Vary layout rhythm across slides (not all same template)
4. Speaker notes: follow Phase 1.5 decision — don't ask again.

## Phase 4 — Verify

1. `/done artifacts/<slug>.html` — opens in browser, checks console, saves screenshot
2. Fix any errors; re-run until clean
3. Invoke `Skill: verify-artifact` silently in background (vision check on layout)
4. Reference `.claude/last-preview.png` in end-of-turn summary

## Phase 5 — Offer next steps

- `/export-pptx artifacts/<slug>.html` — native PPTX via screenshots
- `/export-pdf artifacts/<slug>.html` — PDF
- `/export-standalone artifacts/<slug>.html <dist>.html` — single-file offline HTML
- `/register-asset artifacts/<slug>.html` — add to `assets.html` overview
- Speaker notes mode → add `<script type="application/json" id="speaker-notes">[...]</script>`
- Tweakable mode → `/make-tweakable artifacts/<slug>.html`

## Anti-patterns — hard prohibited

- Title slide first
- Filler content
- AI-slop card patterns (rounded + left border accent)
- Generic gradient backgrounds
- Inventing numbers/stats
- Inventing colors (use `oklch()` to harmonize with existing palette)
