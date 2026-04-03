# Design: Clean Text Toggle

**Date:** 2026-04-03
**Status:** Approved

## Summary

Add a CSS pill-style toggle switch (default off) to the Diffly toolbar that, when enabled, normalises both input texts before running the diff. Normalisation collapses all whitespace and strips punctuation so the comparison focuses purely on word content.

## Context

Diffly (`diffly_app.html`) is a single-file, zero-dependency browser diff tool. Its only existing text pre-processing is `processText(text, mode)`, which optionally strips non-ASCII characters. There is no whitespace or punctuation normalisation. Users who paste text from different sources (PDFs, Word docs, web pages) often get noise diffs caused by formatting differences rather than actual content changes.

## Approach

**Approach B** was selected: a new standalone `cleanText()` utility applied conditionally in `runCompare()`, with a checkbox toggle in the toolbar.

Alternatives considered and rejected:
- Extending `processText()` with a clean flag — pollutes a single-responsibility function.
- Adding a "clean" option to the existing `mode-select` dropdown — conflicts with the user's request for a toggle, conflates encoding and formatting concerns, and cannot be combined with ascii mode.

## UI Component

A CSS pill-style toggle switch in the existing toolbar:

```html
<label class="toggle-label">
  <input type="checkbox" id="clean-toggle">
  <span class="toggle-switch"></span>
  Clean text
</label>
```

- Default: **unchecked (off)**
- Positioned alongside the existing `mode-select`, `granularity-select`, and `view-select` controls
- No external CSS framework — styles implemented inline with existing patterns
- Triggers `scheduleCompare()` on change (same as all other controls)

## `cleanText(text)` Function

New function inserted near `processText()`:

```js
function cleanText(text) {
    return text
        .replace(/[\r\n\t]+/g, ' ')   // line breaks + tabs → space
        .replace(/[^\w\s]/g, '')       // strip punctuation
        .replace(/\s{2,}/g, ' ')       // collapse multiple spaces
        .trim();
}
```

**Operation order matters:** punctuation stripping runs before space collapsing because removing punctuation between words (e.g. `"hello,world"`) can produce adjacent spaces that the subsequent collapse step then cleans up.

Punctuation definition: any character that is not a word character (`\w` = `[a-zA-Z0-9_]`) and not whitespace (`\s`). This strips `.`, `,`, `!`, `?`, `;`, `:`, `-`, `"`, `'`, `(`, `)`, `[`, `]`, `{`, `}`, etc.

## Integration in `runCompare()`

After `processText()`, conditionally apply `cleanText()`:

```js
let textA = processText(leftInput.value, currentMode);
let textB = processText(rightInput.value, currentMode);

if (document.getElementById('clean-toggle').checked) {
    textA = cleanText(textA);
    textB = cleanText(textB);
}
```

The rest of `runCompare()` is unchanged.

## Mirror Overlay Behavior

The existing mirror overlay pattern renders inline diff highlights behind each textarea by matching character positions in the raw input. When clean mode is on, the diff is computed on cleaned text whose character positions no longer correspond to the raw textarea content. Rendering mirrors in this state would produce inaccurate highlights.

**Decision:** When `clean-toggle` is checked, skip `updateMirrors()` and instead clear both mirror divs. The output diff panel continues to work normally and accurately reflects the cleaned text comparison.

## CSS

A minimal CSS pill toggle is added to the `<style>` block:

```css
.toggle-label {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    cursor: pointer;
    font-size: 13px;
    user-select: none;
}
.toggle-label input[type="checkbox"] {
    display: none;
}
.toggle-switch {
    width: 32px;
    height: 18px;
    background: #ccc;
    border-radius: 9px;
    position: relative;
    transition: background 0.2s;
    flex-shrink: 0;
}
.toggle-switch::after {
    content: '';
    position: absolute;
    width: 14px;
    height: 14px;
    background: #fff;
    border-radius: 50%;
    top: 2px;
    left: 2px;
    transition: left 0.2s;
}
.toggle-label input:checked + .toggle-switch {
    background: #4a90e2;
}
.toggle-label input:checked + .toggle-switch::after {
    left: 16px;
}
```

## Files Changed

| File | Change |
|------|--------|
| `diffly_app.html` | Add toggle HTML in toolbar, add `.toggle-label`/`.toggle-switch` CSS, add `cleanText()` function, update `runCompare()` to apply clean step and conditionally skip mirror updates |

## Out of Scope

- Case-insensitive comparison (not requested)
- Per-operation checkboxes (user confirmed single toggle is sufficient)
- Persisting toggle state across sessions
