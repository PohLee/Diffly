# Design: Cleanup Options Panel

**Date:** 2026-04-03
**Status:** Approved

## Summary

Replace the single "Clean text" pill toggle with an expandable "Clean Up" panel below the toolbar containing 6 individual checkboxes (all default off). This gives users granular control over which normalisation steps to apply before diffing.

## Problem

The existing single toggle applies strip-punctuation + flatten-breaks + collapse-spaces + trim all at once. Users need independent control — e.g. stripping punctuation without flattening line breaks, or collapsing whitespace without case folding.

## UI

### Toolbar button
The existing `clean-toggle` label is **removed** from the toolbar and replaced with a `Clean Up ▼` button (`id="cleanup-btn"`). When the panel is open the arrow flips: `Clean Up ▲`. When any checkboxes are checked, a count badge appears inline: `Clean Up (2) ▼`.

### Expandable panel
A `<div id="cleanup-panel">` sits between the toolbar and the summary bar. Hidden by default (`display:none`), shown when the button is clicked. Contains 6 checkboxes in a flex-wrap row:

| ID | Label | Operation |
|----|-------|-----------|
| `opt-strip-punct` | Strip punctuation | Replace `[^\w\s]` with ` ` |
| `opt-flatten-breaks` | Flatten line breaks | Replace `\r\n \n \r` with ` ` (→ single line) |
| `opt-collapse-para` | Collapse paragraph breaks | Replace `\n\n+` with `\n` |
| `opt-collapse-space` | Collapse whitespace | Replace `\s{2,}` with ` ` |
| `opt-case` | Case-insensitive | `text.toLowerCase()` |
| `opt-trim` | Trim whitespace | `text.trim()` |

All default **off**.

## Processing — `applyCleanOpts(text, opts)`

Replaces `cleanText()`. Fixed operation order to avoid artefacts:

```js
function applyCleanOpts(text, opts) {
    if (opts.caseInsensitive)   text = text.toLowerCase();
    if (opts.collapseParaBreaks) text = text.replace(/(\r\n|\r|\n){2,}/g, '\n');
    if (opts.flattenBreaks)     text = text.replace(/\r\n|\r|\n/g, ' ');
    if (opts.stripPunct)        text = text.replace(/[^\w\s]/g, ' ');
    if (opts.collapseSpace)     text = text.replace(/\s{2,}/g, ' ');
    if (opts.trim)              text = text.trim();
    return text;
}
```

Order rationale:
- Lowercase before any splitting so comparisons are consistent.
- Paragraph collapse before flatten — once flatten runs there are no `\n\n` left to match.
- Punctuation replaced with space (not empty) to avoid word merging (e.g. `depreciation,impairment` → `depreciation impairment`).
- Whitespace collapse after punctuation so the newly-inserted spaces get collapsed too.
- Trim last.

## Side effects driven by `opt-flatten-breaks`

| Effect | Condition |
|--------|-----------|
| Disable "Line" granularity option, auto-switch to "Word" | `opt-flatten-breaks` checked |
| Disable Collapse All / Expand All buttons | `opt-flatten-breaks` checked |
| Use `renderCleanOutput()` (inline full-text view) | `opt-flatten-breaks` checked |
| Clear mirror overlays (not update) | **any** option active |

`applyCleanupState()` reads all 6 checkboxes and applies these side effects. Called every time any checkbox changes, before `runCompare()`.

## Badge counter — `updateCleanupBadge()`

Counts checked options. Updates `cleanup-btn` text:
- 0 checked: `Clean Up ▼` / `Clean Up ▲`
- N checked: `Clean Up (N) ▼` / `Clean Up (N) ▲`

## Removals

- `<input id="clean-toggle">` and its `<label class="toggle-label">` in the toolbar HTML
- `.toggle-label`, `.toggle-switch` CSS
- `cleanText()` function
- `applyCleanToggleState()` function
- The `clean-toggle` change event listener

## Files changed

| File | Change |
|------|--------|
| `diffly_app.html` | Remove old toggle HTML/CSS/JS; add cleanup panel HTML + CSS; add `applyCleanOpts()`, `applyCleanupState()`, `updateCleanupBadge()`; update `runCompare()`; wire 6 checkbox listeners + button listener |
