# Clean Text Toggle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a CSS pill-style toggle (default off) that normalises both inputs before diffing — collapsing whitespace, stripping punctuation — and clears mirror overlays when active.

**Architecture:** Single file (`diffly_app.html`) modified in four places: CSS block (toggle styles), HTML toolbar (toggle element), JS utility section (`cleanText` function), and `runCompare` (conditional clean + mirror logic). No new files.

**Tech Stack:** Vanilla JS, HTML5, CSS3 — no dependencies, no build step.

---

### Task 1: Add CSS for the pill toggle switch

**Files:**
- Modify: `diffly_app.html:333-338` (end of `<style>` block, before `</style>`)

- [ ] **Step 1: Add toggle CSS inside the `<style>` block, immediately before the closing `</style>` tag (line 339)**

Insert the following block right before `</style>`:

```css
        /* ── Clean-text toggle switch ── */
        .toggle-label {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            cursor: pointer;
            font-size: 0.85rem;
            font-weight: 500;
            color: #374151;
            user-select: none;
            white-space: nowrap;
        }
        .toggle-label input[type="checkbox"] { display: none; }
        .toggle-switch {
            width: 32px;
            height: 18px;
            background: #d1d5db;
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
            box-shadow: 0 1px 2px rgba(0,0,0,0.15);
        }
        .toggle-label input:checked + .toggle-switch { background: #2563eb; }
        .toggle-label input:checked + .toggle-switch::after { left: 16px; }
```

- [ ] **Step 2: Verify the CSS renders correctly**

Open `diffly_app.html` in a browser. The toolbar should still look intact (no visual change yet — the toggle element hasn't been added). No console errors.

---

### Task 2: Add the toggle HTML to the controls toolbar

**Files:**
- Modify: `diffly_app.html:367-368` (after `view-select` control-group, before `compare-btn`)

- [ ] **Step 1: Add the toggle label element in the `.controls` div**

Insert the following after the closing `</div>` of the `view-select` control-group (line 367) and before `<button class="compare-btn"`:

```html
        <label class="toggle-label">
            <input type="checkbox" id="clean-toggle">
            <span class="toggle-switch"></span>
            Clean text
        </label>
```

- [ ] **Step 2: Verify the toggle appears in the browser**

Open `diffly_app.html`. The toolbar should now show: `Mode | Granularity | View | [Clean text toggle] | [Compare button]`. The toggle should be in the off (grey) state by default. Clicking it should flip to blue. No console errors.

---

### Task 3: Add `cleanText()` utility function

**Files:**
- Modify: `diffly_app.html:541` (after `processText()`, before the diff engine section)

- [ ] **Step 1: Insert `cleanText()` immediately after `processText()` (after line 541)**

```js
function cleanText(text) {
    return text
        .replace(/[\r\n\t]+/g, ' ')   // line breaks + tabs → space
        .replace(/[^\w\s]/g, '')       // strip punctuation
        .replace(/\s{2,}/g, ' ')       // collapse multiple spaces
        .trim();
}
```

- [ ] **Step 2: Verify the function in the browser console**

Open browser DevTools console and run:

```js
cleanText("Hello,  world!\nThis is a test.\t\tDone.")
```

Expected output: `"Hello world This is a test Done"`

Also test edge cases:
```js
cleanText("")          // → ""
cleanText("   ")       // → ""
cleanText("a,b.c!d")   // → "abcd"
```

---

### Task 4: Update `runCompare()` to apply clean step and handle mirrors

**Files:**
- Modify: `diffly_app.html:835-860` (`runCompare` function)

- [ ] **Step 1: Replace the diff computation and mirror update lines in `runCompare()`**

Current code (lines 850-856):
```js
    const mode        = document.getElementById('mode-select').value;
    const granularity = document.getElementById('granularity-select').value;
    const viewMode    = document.getElementById('view-select').value;

    const diff = computeDiff(processText(textA, mode), processText(textB, mode), granularity);

    updateMirrors(diff, granularity);
```

Replace with:
```js
    const mode        = document.getElementById('mode-select').value;
    const granularity = document.getElementById('granularity-select').value;
    const viewMode    = document.getElementById('view-select').value;
    const doClean     = document.getElementById('clean-toggle').checked;

    let procA = processText(textA, mode);
    let procB = processText(textB, mode);
    if (doClean) {
        procA = cleanText(procA);
        procB = cleanText(procB);
    }

    const diff = computeDiff(procA, procB, granularity);

    if (doClean) {
        clearMirrors();
    } else {
        updateMirrors(diff, granularity);
    }
```

- [ ] **Step 2: Wire the clean toggle to trigger runCompare**

At the bottom of the script (after line 873, where the other event listeners live), add:

```js
document.getElementById('clean-toggle').addEventListener('change', runCompare);
```

- [ ] **Step 3: Functional verification in the browser**

Test scenario 1 — toggle OFF (default behaviour unchanged):
1. Paste `Hello\nWorld` in Text A, `Hello World` in Text B
2. Toggle is off → diff shows a difference (line break vs space)
3. Mirror highlights are visible on both sides

Test scenario 2 — toggle ON (clean mode):
1. Same inputs
2. Enable "Clean text" toggle
3. Diff should show **identical** (both normalise to `"Hello World"`)
4. Mirror overlays should be **cleared** (no highlights in textareas)
5. Summary bar shows "Exact match"

Test scenario 3 — punctuation stripping:
1. Paste `Hello, world! How are you?` in Text A, `Hello world How are you` in Text B
2. Toggle OFF → diff shows deletions (commas, exclamation, question mark)
3. Toggle ON → diff shows identical

Test scenario 4 — toggle reactivity:
1. Enter differing text, toggle ON → shows clean diff
2. Toggle back OFF → mirrors reappear and original diff resumes
3. Auto-compare fires correctly (no stale state)

- [ ] **Step 4: Commit**

```bash
git add diffly_app.html
git commit -m "feat: add clean text toggle to normalise whitespace and strip punctuation before diff"
```
