---
name: accessibility-audit
description: >
  Audits a UI component or page for WCAG 2.2 AA compliance and produces a severity-ranked report.
  Use when the user asks to check accessibility, audit a11y, review WCAG compliance, check contrast,
  verify keyboard support, review focus management, or mentions screen readers, assistive technology,
  or inclusive design. Handles source code (JSX/HTML + CSS), live URLs, and screenshots as input.
---

# Accessibility Audit

Audits a UI component or page for WCAG 2.2 AA compliance and produces a severity-ranked findings report. Use after building or updating any user-facing UI, or when preparing for an accessibility review.

## Purpose

Accessibility issues are the bugs that affect the most people while being the easiest to miss during visual development. A sighted developer using a mouse will never notice a missing focus indicator, a broken tab order, or a contrast ratio that fails by 0.3 points. This skill provides a structured, repeatable audit that catches those gaps before they reach users — or an external auditor.

The audit targets **WCAG 2.2 AA** and is token-aware: it checks whether colour values use design tokens, resolves them to their computed values for contrast calculations, and flags hardcoded colours that bypass the token system.

The audit accepts either of these as input:

1. **Source code** — component files (JSX/TSX/HTML + CSS/Tailwind/styled-components), including any associated token definitions or theme files
2. **Live URL or screenshot** — a rendered page or component, with or without dev tools context

If the input is a screenshot only, note which checks can't be performed (keyboard, focus, ARIA) rather than skipping silently — the user needs to know what wasn't covered.

## Process

1. **Check document structure and semantics.** Verify the HTML uses appropriate landmark elements (`<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`) and that the heading hierarchy is logical and unbroken (no skipping from `h2` to `h4`). Check that lists use `<ul>`/`<ol>`/`<dl>`, tables use `<table>` with proper `<th>` and `scope`, and interactive elements use their native HTML equivalents — `<button>` not `<div onClick>`, `<a>` not `<span role="link">`. Native elements carry built-in keyboard support, focus management, and screen reader semantics for free, so every custom replacement is a liability unless it fully replicates that behaviour. Verify `<html lang="...">` is present and correct (SC 3.1.1, Level A) — screen readers depend on this to select the correct pronunciation engine.

2. **Audit ARIA usage.** Check that ARIA roles, states, and properties are used correctly and only where native HTML can't achieve the same result — ARIA's first rule is to not use ARIA if a native element will do. Verify that `aria-label`, `aria-labelledby`, and `aria-describedby` references point to elements that exist. Check that dynamic state changes (`aria-expanded`, `aria-selected`, `aria-checked`, `aria-disabled`, `aria-live`) update correctly in response to interactions. Flag `role="presentation"` or `aria-hidden="true"` on elements that contain focusable children, as this hides content from assistive technology while leaving a keyboard trap.

3. **Evaluate colour contrast.** For every text and interactive UI element, calculate the contrast ratio between foreground and background. The thresholds are 4.5:1 for normal text (under 24px or 18.66px bold) and 3:1 for large text and UI components (WCAG 1.4.3, 1.4.11). Where values reference design tokens, resolve the token to its computed colour and check the ratio. Where values are hardcoded, flag both the contrast result and the missing token — a hardcoded `#666` that passes today will break silently if the background token changes. If checking from a screenshot, use visual estimation and note the result as approximate.

4. **Test keyboard navigation.** Verify that every interactive element (links, buttons, inputs, selects, custom controls) is reachable via Tab and operable via Enter, Space, or arrow keys as appropriate. Check that tab order follows a logical reading flow and doesn't jump unpredictably. Verify that no keyboard traps exist — the user must be able to Tab into and out of every component, including modals, dropdowns, and date pickers. For modal dialogs, confirm that focus is trapped within the modal while open and restored to the triggering element on close. Check for skip navigation links on page-level audits. If working from source code only, trace `tabindex` values and flag any positive `tabindex` (which forces an unnatural tab order) and any missing `tabindex` on custom interactive elements.

5. **Review focus management and indicators.** Confirm every focusable element has a visible focus indicator that meets WCAG 2.4.7 (visible) and ideally 2.4.11 (focus not obscured by other content). Check that `:focus-visible` is used rather than `:focus` to avoid showing focus rings on mouse clicks. Verify focus indicators have sufficient contrast — a 3:1 ratio against adjacent colours. Check that focus is programmatically managed during dynamic content changes: when a section expands, a drawer opens, or content loads asynchronously, focus should move to the new content or a logical anchor. Flag any `outline: none` or `outline: 0` without a replacement indicator.

6. **Check touch and pointer targets.** Verify interactive targets meet the WCAG 2.2 minimum size of 24×24 CSS pixels (SC 2.5.8), with adequate spacing between adjacent targets. Flag targets that rely solely on dragging — WCAG 2.2 SC 2.5.7 requires a non-dragging alternative for every drag interaction. Check that hover-triggered content (tooltips, menus) is also accessible via focus and is dismissable, hoverable, and persistent per WCAG 1.4.13.

7. **Verify images, media, and motion.** Check that all `<img>` elements have `alt` attributes — meaningful alt text for informative images, empty `alt=""` for decorative ones. Verify that SVG icons used as interactive elements have accessible names via `aria-label` or `<title>`. Flag any auto-playing media without pause controls. Check for `prefers-reduced-motion` support on any animations or transitions — users who experience motion sickness or vestibular disorders need a way to disable non-essential movement. If the component includes video, check for captions or transcript support.

8. **Check forms and error handling.** Verify that every form input has a visible, associated `<label>` (via `for`/`id` or wrapping). Check that required fields are indicated both visually and programmatically (`aria-required` or `required` attribute). Verify error messages are associated with their inputs via `aria-describedby` and are announced to screen readers (via `aria-live` or focus management). Check that error messages are specific — "This field is required" rather than a generic "Error". Check that common personal data fields (`name`, `email`, `tel`, `address`, etc.) have appropriate `autocomplete` attributes (SC 1.3.5, Level AA). For WCAG 2.2 AAA (note: beyond the default AA target), check that the form doesn't require the user to re-enter information they've already provided (SC 3.3.7) and that authentication doesn't rely on cognitive function tests (SC 3.3.8) — only report these if the user has requested AAA coverage.

## Output Format

Return a summary block followed by a findings table.

**Summary block:**

```
Component/Page: [name]
Input type: [source code | live URL | screenshot]
WCAG target: 2.2 AA
Token-aware: [yes — tokens resolved | no — raw values only | partial — some tokens unresolvable]
Overall compliance: [Pass | Conditional Pass | Fail]
Issues: [X critical, Y serious, Z moderate, W minor]
Checks not performed: [list any checks skipped due to input limitations]
```

**Findings table:**

| #   | Phase | WCAG SC | Severity | Element | Issue | Recommendation |
| --- | ----- | ------- | -------- | ------- | ----- | -------------- |

Number each finding for easy reference in follow-up conversations.

Severity levels:

- 🔴 **Critical** — blocks access entirely for some users (missing keyboard access, no alt text on essential images, contrast below 3:1, keyboard trap)
- 🟠 **Serious** — causes significant difficulty (contrast between 3:1 and 4.5:1 on normal text, missing form labels, focus not managed on dynamic content)
- 🟡 **Moderate** — creates friction but has workarounds (missing skip link, heading hierarchy broken, positive tabindex, touch target slightly under 24px)
- 🔵 **Minor** — best practice or enhancement (decorative image with non-empty alt, redundant ARIA on native element, missing `prefers-reduced-motion`)

Include the specific WCAG success criterion (e.g. SC 1.4.3, SC 2.5.8) in every finding so the user can look up the normative requirement. If a finding relates to design token usage rather than a WCAG criterion, use "Token" in the WCAG SC column.

If a phase produces no findings, include a single row noting the phase passed — an explicit "no issues" prevents the user wondering whether the check ran.

## Rules

- This is a read-only audit — do not modify any files
- Target WCAG 2.2 AA unless the user specifies a different level — if they ask for AAA, note which criteria are AAA so they can prioritise
- When checking contrast, resolve design tokens to their computed values before calculating ratios — report both the token name and the resolved hex value so the finding is actionable
- If a colour is hardcoded, flag it twice: once for the contrast result (pass or fail) and once for the missing token — these are separate concerns with different fixes
- When working from a screenshot, mark contrast findings as "approximate" and keyboard/focus/ARIA findings as "not verifiable from screenshot" — do not guess at behaviour you can't observe
- Do not flag native HTML elements for missing ARIA when the native semantics are sufficient — adding redundant ARIA (e.g. `role="button"` on `<button>`) is noise, not an improvement
- Group related issues — if the same pattern repeats across multiple instances (e.g. five buttons all missing visible focus), report it once with a scope note rather than five separate rows
- Account for CSS inheritance and cascade when checking styles — a focus indicator might be set globally rather than on each component
- Distinguish between issues that block compliance (must fix) and enhancements that improve the experience (should fix) — the severity levels exist for this reason, so use them precisely
