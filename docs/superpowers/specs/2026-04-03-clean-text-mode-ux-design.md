# Design: Clean Text Mode UX — Granularity Lock + Inline Output

**Date:** 2026-04-03
**Status:** Approved

## Problem

When "Clean text" is on, all whitespace is normalised into a single line. Two UX breakages follow:

1. **Granularity "Line"** is nonsensical — the entire input is one line, so line-level diff produces either "identical" or "completely different" with no useful signal.
2. **Word granularity output** (current chunked side-by-side rendering) strips context — showing only the changed words isolated from their surrounding text makes it hard to understand what changed and where.

## Solution

### Fix 1: Disable "Line" granularity when clean text is on

- The "Line" `<option>` in `#granularity-select` gets `disabled` when `#clean-toggle` is checked.
- If "Line" was the active selection at toggle time, auto-switch to "Word".
- The option stays **visible** (not hidden) so the user understands it exists but is unavailable in this mode.
- Re-enabled when clean text is turned off.
- Implemented in a helper `applyCleanToggleState()` called from the clean toggle's change handler.

### Fix 2: New inline full-text output rendering for clean text mode

Replace the standard chunked `renderOutput()` call with `renderCleanOutput(diff, viewMode)` when clean is on.

**Side-by-side view:**
- Left panel: full Text A rendered as one paragraph. Deleted words get `<span class="deletion">`. Unchanged words rendered as plain text.
- Right panel: full Text B rendered as one paragraph. Added words get `<span class="addition">`. Unchanged words rendered as plain text.
- Both panels use existing `.chunk-left` / `.chunk-right` / `.chunk-content` styles inside a new `.clean-sbs` grid container.

**Unified view:**
- Single panel with the merged text. Unchanged words plain, deletions in red, additions in green — all inline in one flow. Uses a new `.clean-unified` container.

**Nav controls behaviour:**
- Prev/Next diff: naturally disabled because `collectChunks()` finds no `.diff-chunk` elements.
- Collapse All / Expand All: explicitly disabled (`disabled` attribute) in clean mode; re-enabled when clean is off.

**Summary bar:** unchanged — still displays deletion/addition counts.

## CSS additions

```css
.clean-sbs {
    display: grid;
    grid-template-columns: 1fr 1fr;
}
.clean-unified {
    padding: 6px 10px;
}
@media (max-width: 768px) {
    .clean-sbs { grid-template-columns: 1fr; }
}
```

`.chunk-left`, `.chunk-right`, `.chunk-content`, `.deletion`, `.addition` are all reused unchanged.

## Files changed

| File | Change |
|------|--------|
| `diffly_app.html` | Add `applyCleanToggleState()`, add `renderCleanOutput()`, update `runCompare()`, add `.clean-sbs` / `.clean-unified` CSS, disable/re-enable collapse buttons |

## Out of scope

- Saving/restoring previous granularity when clean is toggled off
- Changing the character granularity option (still valid in clean mode)
