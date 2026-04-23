# Firefox Android Viewport Fix Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make all shared mobile bottom-sheet modals size themselves to the visible viewport so Firefox Android keyboard open does not hide action buttons below reachable scroll.

**Architecture:** Extract viewport sizing into a shared frontend helper that writes a CSS custom property based on `visualViewport` with `innerHeight` fallback. Use one reusable mobile sheet class in the shared layout and replace hard-coded mobile `vh` modal limits in list and home templates.

**Tech Stack:** Go templates, plain browser JavaScript, CSS in `templates/layout.html`, Node built-in test runner, Go test

---

### Task 1: Add regression coverage for viewport sizing helper

**Files:**
- Create: `test/viewport-height.test.js`
- Create: `static/viewport.js`

- [ ] **Step 1: Write the failing test**

Create `test/viewport-height.test.js` with assertions for:
- `getVisibleViewportHeight` prefers `visualViewport.height`
- fallback uses `innerHeight`
- `syncViewportHeight` writes `--app-viewport-height` in `px`

- [ ] **Step 2: Run test to verify it fails**

Run: `node --test test/viewport-height.test.js`
Expected: FAIL because `static/viewport.js` does not exist yet.

- [ ] **Step 3: Write minimal implementation**

Create `static/viewport.js` with exported helper functions and browser initialization hooks.

- [ ] **Step 4: Run test to verify it passes**

Run: `node --test test/viewport-height.test.js`
Expected: PASS

### Task 2: Wire shared viewport CSS and initialization

**Files:**
- Modify: `templates/layout.html`

- [ ] **Step 1: Add shared mobile sheet CSS**

Define CSS variables and a reusable `.mobile-sheet` class that:
- uses `max-height: calc(var(--app-viewport-height, 100vh) - 1rem)`
- keeps `overflow-y: auto`
- preserves safe-area bottom padding
- limits changes to mobile only, leaving desktop classes in place

- [ ] **Step 2: Load viewport helper globally**

Add a script include for `/static/viewport.js` before `/static/app.js` and initialize it from the shared layout so every page gets the same viewport syncing behavior.

### Task 3: Replace hard-coded mobile sheet height usage

**Files:**
- Modify: `templates/list.html`
- Modify: `templates/home.html`

- [ ] **Step 1: Replace shared bottom-sheet panel sizing**

Update all mobile bottom-sheet modal panels that currently use `max-h-[90vh]` or fixed mobile height assumptions to also use `.mobile-sheet`.

- [ ] **Step 2: Keep desktop behavior unchanged**

Preserve existing `md:max-w-*`, `md:max-h-*`, `md:rounded-*`, and flex behavior for desktop layouts.

### Task 4: Verify end to end

**Files:**
- Test: `test/viewport-height.test.js`
- Test: Go package tree

- [ ] **Step 1: Run focused frontend regression test**

Run: `node --test test/viewport-height.test.js`
Expected: PASS

- [ ] **Step 2: Run backend/template verification**

Run: `go test ./...`
Expected: PASS

- [ ] **Step 3: Inspect final coverage**

Check that list and home mobile sheet instances use `.mobile-sheet` and no target modal still depends on `90vh`.
