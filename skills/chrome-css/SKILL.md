---
name: chrome-css
description: read computed CSS styles and layout metrics from page elements via Chrome DevTools. Use when you need fontSize, colors, margins, padding, box model, or bounding rects — anything beyond DOM structure. Wraps into clean patterns.
allowed-tools: mcporter read
---

# Chrome CSS — Computed Styles & Layout

Reads CSS computed styles and box-model metrics from the browser.
Wraps `` into predictable, token-efficient calls.

## When to use this skill

- "What font/color/size is this element?"
- "What are the exact margins/padding?"
- "Is this element visible? What's its position?"
- "Compare styles of two elements"
- Any time you'd write `getComputedStyle()` or `getBoundingClientRect()`

## NOT for these (use native tools instead)

- Reading text content → use `` (already includes text)
- Finding elements by tag/role → use ``
- Getting element attributes → use ``
- Clicking, filling, hovering → use `click`, `fill`, `fill_form`, `hover`

## How to call

All calls use `` via mcporter. Each pattern below shows the exact JS snippet.

### getComputedStyles(uid) — all computed CSS

```
mcporter(action="call" selector="mcporter "
 args={"function": "(el) => { const s = getComputedStyle(el); const out = {}; for (let p of s) { out[p] = s.getPropertyValue(p); } return out; }"
 "args": ["<uid from snapshot>"]})
```

### getComputedStyles(uid, [properties]) — specific properties only

```
mcporter(action="call" selector="mcporter "
 args={"function": "(el, props) => { const s = getComputedStyle(el); const out = {}; props = JSON.parse(props); props.forEach(p => { out[p] = s.getPropertyValue(p); }); return out; }"
 "args": ["<uid>", "[\"fontSize\",\"color\",\"backgroundColor\",\"display\",\"position\",\"margin\",\"padding\"]"]})
```

### getBoxModel(uid) — sizes and positions

```
mcporter(action="call" selector="mcporter "
 args={"function": "(el) => { const r = el.getBoundingClientRect(); const s = getComputedStyle(el); return { x, y, width: r.width, height: r.height, top: r.top, left: r.left, margin: { top: s.marginTop, right: s.marginRight, bottom: s.marginBottom, left: s.marginLeft }, padding: { top: s.paddingTop, right: s.paddingRight, bottom: s.paddingBottom, left: s.paddingLeft }, border: { top: s.borderTopWidth, right: s.borderRightWidth, bottom: s.borderBottomWidth, left: s.borderLeftWidth } }; }"
 "args": ["<uid>"]})
```

### isVisible(uid) — visibility check

```
mcporter(action="call" selector="mcporter "
 args={"function": "(el) => { const s = getComputedStyle(el); const r = el.getBoundingClientRect(); return { visible: s.display !== 'none' && s.visibility !== 'hidden' && parseFloat(s.opacity) > 0 && r.width > 0 && r.height > 0, display: s.display, visibility: s.visibility, opacity: s.opacity, width: r.width, height: r.height }; }"
 "args": ["<uid>"]})
```

### batchGetStyles(uids[]) — multiple elements at once

```
mcporter(action="call" selector="mcporter "
 args={"function": "() => { const uids = JSON.parse(arguments[0]); return uids.map(uid => { const el = document.querySelector('[data-uid=\"' + uid + '\"]') || document.querySelector('#' + uid) || (() => { throw new Error('Cannot find element by uid: ' + uid); })(); const s = getComputedStyle(el); return { uid, tag: el.tagName, fontSize: s.fontSize, color: s.color, display: s.display }; }); }"
 "args": ["[\"3_12\",\"3_28\"]"]})
```

## Anti-patterns (avoid these)

- ❌ `document.body.innerText` → use `` (already has all text)
- ❌ `document.querySelectorAll('a')` → use `` (already has DOM tree)
- ❌ `el.outerHTML` → use `` (verbose mode for full structure)
- ❌ Writing a new `` from scratch for CSS → use patterns above
