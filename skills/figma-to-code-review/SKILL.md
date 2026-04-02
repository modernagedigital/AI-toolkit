---
name: figma-to-code-review
description: >
  Reviews a coded UI component against its Figma source design and produces a structured fidelity report.
  Use this skill whenever the user asks to compare a Figma design to code, check implementation fidelity,
  audit design-to-code accuracy, review a component against its design spec, or mentions visual drift
  between design and implementation. Also trigger when the user provides both a Figma reference and
  component code and asks for a review, comparison, or audit — even if they don't use the word "Figma"
  explicitly (e.g. "does this match the design?", "check this against the mockup", "is my component
  faithful to the spec?").
---

# Figma-to-Code Review

Reviews a coded component against its Figma source design and produces a structured fidelity report. Use after implementing or updating any UI component that has a Figma counterpart.

## Compatibility

Requires at least one Figma MCP server (official or console) to extract structured design data. Without MCP access, this skill cannot run — do not attempt a review from screenshots alone.

## Purpose

Design-to-code drift is cumulative. A hardcoded colour here, a missing hover state there — individually minor, but over time they erode trust in the design system and create maintenance debt. This skill catches that drift early by walking through a structured comparison across tokens, spacing, typography, responsive behaviour, variant coverage, and accessibility.

The review requires two inputs:

1. **Figma reference** — a Figma file/frame URL. Extract structured data (properties, variants, tokens, constraints) via MCP tools — see "Which MCP tools to use" below. **Screenshots must never be used to infer measurements, token values, spacing, or any spec data.** Models hallucinate dimensions from pixels and the results are unreliable. If structured inspect data is unavailable for a node, mark those properties as "Unverifiable" and tell the user — do not fall back to eyeballing a screenshot. Screenshots may be captured *alongside* structured data as a visual sanity-check, but they are reference only — every value in the report must come from MCP inspect data or the code itself.
2. **Code implementation** — the component source file(s) (JSX/TSX/HTML + CSS/Tailwind/styled-components), a Storybook story. If the component is used in multiple files (e.g. a base component and a theme wrapper), include all relevant files to get a complete picture of the implementation.

### Which MCP tools to use

Pick based on what you need:

- **`get_design_context`** (official Figma MCP) — best starting point. Returns code hints, tokens, properties, and a screenshot in one call. Use this first for most reviews.
- **`get_metadata`** (official Figma MCP) — for file-level info when you need page structure or component inventory.
- **`figma_get_component_details`** / **`figma_get_styles`** (console MCP) — deeper component inspection: full variant matrices, style definitions, nested layer data.
- **`figma_get_variables`** / **`figma_get_token_values`** (console MCP) — when you need to verify exact token names and resolved values.
- **`figma_take_screenshot`** / **`get_screenshot`** — visual reference only. Useful for confirming layout intent at a glance, but never use as a source for measurements or token values.

If either input is missing, ask the user for it before proceeding — partial reviews produce misleading confidence.

## Process

1. **Extract Figma data.** Before comparing anything, extract all structured data from the Figma node using MCP tools. Follow the checklist in `scripts/extraction-checklist.md` to make sure nothing is missed. This upfront extraction prevents back-and-forth MCP calls mid-review and ensures you have complete data before forming judgements.

2. **Map structure.** Compare the Figma layer hierarchy against the DOM/component tree. Check that semantic HTML elements match the design intent — a Figma "Button" layer should map to a `<button>`, not a styled `<div>`. Verify that slot and children composition (icon + label, leading/trailing elements) mirrors the Figma layer structure. Flag anything present in one but missing from the other. This matters because structural mismatches cause accessibility and maintainability problems that are harder to fix later.

   **Component composition:** When the Figma component contains nested sub-components (e.g. a Card containing a Button), audit the target component's direct properties and flag mismatches at the interface boundary (wrong variant prop passed, missing slot, incorrect nesting order). Don't recurse into the internals of nested components unless the user specifically asks — those components should get their own review.

3. **Audit design tokens.** Extract every visual value from the code — colours, spacing, border-radius, shadows, font sizes, font weights, line heights, z-indices. For each value, check whether it references a design token or is hardcoded. Where tokens are used, verify the token name matches the Figma specification. Consult `references/token-mapping-guide.md` for common translation patterns across ecosystems (CSS custom properties, Tailwind, styled-components, etc.). Hardcoded values that resolve to the correct visual output are still a problem because they break when tokens are updated — flag these as warnings, not criticals.

   **Abstraction layers:** Many projects wrap design tokens in an intermediate mapping — Figma token → Tailwind config → utility class, or Figma token → theme provider → CSS custom property. When this exists, check the full chain: does the Figma token map to the right config entry, and does the code reference that config entry correctly? A value can look correct in the browser but point to the wrong token underneath.

4. **Check spacing and sizing.** Compare padding, margin, and gap values between Figma and code. Consult `references/constraint-mapping.md` for the Figma constraint → CSS equivalents reference. Verify min/max constraints and check that alignment and distribution match (Figma auto-layout properties ↔ flexbox/grid). Spacing inconsistencies are the most common source of "it looks slightly off" feedback from designers, so be precise here.

5. **Compare typography.** Check font family, size, weight, line height, letter spacing, and text transform. Verify that text truncation and overflow behaviour matches — Figma's text truncation should correspond to `text-overflow: ellipsis`, `-webkit-line-clamp`, or equivalent. Check text alignment and wrapping. Typography mismatches are visually obvious and undermine confidence in the implementation.

6. **Verify variant and state coverage.** List every Figma variant (size, colour, theme) and confirm each has a corresponding code implementation. Check interactive states: hover, focus-visible, active, disabled, loading, error. Flag code variants that don't exist in Figma (implementation drift) and Figma variants missing from code (incomplete implementation). Missing states are a common source of bugs reported after launch.

7. **Test responsive behaviour.** If Figma includes responsive breakpoints or layout shifts, verify the code handles them. Check flex-wrap, grid breakpoints, and container queries against the intended responsive behaviour. If Figma uses different spacing or sizing values at different breakpoints, confirm the code follows suit. Skip this step if the Figma source is a single fixed-size frame with no responsive annotations.

8. **Check accessibility.** Verify interactive elements are keyboard accessible and have appropriate ARIA attributes. Confirm visible focus indicators exist (`:focus-visible`). Check colour contrast against WCAG AA: 4.5:1 for normal text, 3:1 for large text and UI components. Verify decorative elements are hidden from assistive technology (`aria-hidden`, empty alt text). Accessibility gaps caught here prevent entire categories of bug reports.

## Output Format

Return a summary block followed by a findings table.

**Summary block:**

```
Component: [name]
Figma source: [URL or description]
Code source: [file path(s)]
Overall fidelity: [High | Medium | Low]
Issues: [X critical, Y warning, Z info]
```

**Findings table:**

| Phase | Severity | Property | Figma Value | Code Value | Recommendation |
| ----- | -------- | -------- | ----------- | ---------- | -------------- |

Severity levels:

- 🔴 **Critical** — visible mismatch, accessibility failure, or missing variant used in production layouts
- 🟡 **Warning** — hardcoded value that should be a token, minor spacing discrepancy, or missing edge-case state
- 🔵 **Info** — naming inconsistency, suggestion for improvement, or implementation that works but could be more robust

If no issues are found in a phase, note it as passing in the summary rather than omitting it — an explicit "no issues" gives the user confidence the check actually ran.

## Rules

- This is a read-only audit — do not modify any files
- If a Figma value and code value resolve to the same visual result but one uses a token and the other doesn't, flag as warning not critical — the output is correct today, but the maintenance risk is real when tokens change
- If a value can't be determined from the Figma input (e.g. you cannot get the inspect data), mark it "Unverifiable" rather than guessing — a wrong assumption is worse than a gap
- Account for CSS inheritance and cascade when reviewing — a value might not be declared on the component itself but inherited from a parent or set via a utility class
- Always check both the default/rest state and at least one interactive state (hover or focus) — if only a static screenshot is provided, note which states could not be verified
- Group related issues — if the same token is hardcoded in five places, report it once with a note about scope rather than five separate rows, to keep the report actionable
- Do not flag intentional platform adaptations (e.g. a Figma mockup showing a macOS-specific control that renders differently on the web)

## What this skill does NOT do

- **Generate code fixes.** The output is a report — it does not modify any files or produce corrected code. The user or another skill handles remediation.
- **Review animation or motion specs.** Figma prototyping transitions are not reliably extractable via MCP, so motion fidelity is out of scope.
- **Deep-audit nested sub-components.** When a component contains other components, this skill checks the interface boundary (correct variant, correct slot usage) but does not recurse into the nested component's internals. Each component should get its own review.
