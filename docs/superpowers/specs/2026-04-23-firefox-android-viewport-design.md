# Firefox Android Viewport Fix Design

**Date:** 2026-04-23
**Issue:** [PanSalut/Koffan#109](https://github.com/PanSalut/Koffan/issues/109)

## Problem

On Firefox for Android, opening the software keyboard inside mobile bottom-sheet forms can leave the action buttons below the visible viewport. The current implementation relies on fixed-position mobile modals with height constraints like `max-h-[90vh]`, which are based on layout viewport assumptions. On mobile browsers, especially when the keyboard opens, the visible viewport can shrink independently from the layout viewport. As a result, the modal remains too tall and the user cannot scroll far enough to reach the bottom controls.

This is not limited to the "add item" modal. The same bottom-sheet pattern is reused across multiple mobile modals in `templates/list.html` and `templates/home.html`, so a local workaround would leave the same bug class elsewhere.

## Decision

Implement a shared viewport sizing mechanism for all mobile bottom-sheet modals.

The system will:

- compute the visible viewport height from `window.visualViewport.height` when available
- fall back to `window.innerHeight` when `visualViewport` is unavailable
- expose the computed value through a CSS custom property on `document.documentElement`
- replace hard-coded mobile modal height limits such as `90vh` with a shared class that derives max height from the custom property
- keep desktop modal sizing unchanged

## Why This Approach

### Option 1: Local `scrollIntoView()` workarounds on focused inputs

Rejected. This would treat the symptom in one or two forms, but would not fix the shared bottom-sheet sizing problem. It would also be browser-fragile and likely require repeated exceptions.

### Option 2: Pure CSS switch from `vh` to `dvh` or `svh`

Rejected as the primary fix. New viewport units help, but browser behavior around on-screen keyboards is still inconsistent enough that CSS alone is not a reliable cross-browser fix for this specific issue.

### Option 3: Shared JS + CSS viewport contract

Accepted. This addresses the root cause at the layout boundary and gives one reusable fix for every mobile bottom-sheet modal.

## Scope

### In scope

- global viewport-height updater in frontend JavaScript
- shared CSS variables and modal utility classes in the base layout
- replacement of mobile modal height constraints in list and home templates
- mobile-only behavior changes

### Out of scope

- desktop modal redesign
- unrelated modal structure refactors
- browser-specific hacks per individual form

## Implementation Design

### 1. Shared viewport custom property

Add a small frontend helper that updates a CSS variable such as `--app-viewport-height` on:

- initial page load
- `resize`
- `orientationchange`
- `visualViewport.resize`
- `visualViewport.scroll`

Using `visualViewport.scroll` matters because some browsers move the visible viewport when the keyboard opens without a classic layout resize.

### 2. Shared bottom-sheet sizing class

Define a reusable CSS class for mobile bottom-sheet panels, for example `.mobile-sheet`.

Mobile behavior:

- width remains full on mobile
- `max-height` is derived from `--app-viewport-height`
- panel remains scrollable with `overflow-y: auto`
- bottom safe-area padding remains supported

Desktop behavior:

- preserve current `md:max-w-*`, `md:rounded-*`, and desktop max-height behavior

### 3. Template updates

Replace current mobile `max-h-[90vh]` usage in shared modal panels with the new class, covering:

- list page modals in `templates/list.html`
- home page modals in `templates/home.html`

The class should be applied only where the component behaves as a mobile bottom-sheet. Existing non-sheet elements should not be changed.

## Risks

### Risk: viewport updates firing too often

Mitigation: keep the updater extremely small and write only one CSS custom property. No DOM queries beyond `document.documentElement.style.setProperty`.

### Risk: desktop regressions

Mitigation: scope the new sizing rules to mobile by default and keep existing desktop utility classes intact.

### Risk: incomplete coverage

Mitigation: update every modal using the same bottom-sheet pattern now, not only the issue reproducer.

## Verification

Because the repository does not currently include browser E2E coverage for this UI, verification will be:

- static code verification that all shared mobile sheet instances use the new class
- `go test ./...` to ensure no backend regressions from template changes
- manual reasoning check that the viewport helper is initialized from the shared layout and therefore covers both list and home pages

## Success Criteria

- mobile bottom-sheet modals no longer depend on fixed `90vh` assumptions
- Firefox Android can shrink the visible viewport after keyboard open without hiding submit buttons below the reachable scroll area
- the same fix applies consistently across all shared mobile bottom-sheet modals
