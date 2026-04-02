# Figma Data Extraction Checklist

Run through this checklist before starting the comparison. Extract all data upfront to avoid repeated MCP calls mid-review.

## 1. Component identity

- [ ] Component name and description
- [ ] File key and node ID (from the URL)
- [ ] Page and frame location within the file

## 2. Variants

- [ ] All variant properties and their values (e.g. Size: sm/md/lg, State: default/hover/disabled)
- [ ] Default variant combination
- [ ] Total variant count (properties x values)

## 3. Design tokens

- [ ] Colour tokens referenced (fills, strokes, text colours)
- [ ] Spacing tokens (padding, gap, margin — if tokenised)
- [ ] Typography tokens (font family, size, weight, line height, letter spacing)
- [ ] Border radius tokens
- [ ] Shadow/elevation tokens
- [ ] Opacity values
- [ ] Z-index or layer ordering (if relevant)

For each token, note both the **token name** and the **resolved value** — you need both to verify the code references the right token and that it resolves correctly.

## 4. Layout and constraints

- [ ] Auto-layout direction (horizontal/vertical)
- [ ] Auto-layout gap value
- [ ] Padding (all four sides — note if symmetric or asymmetric)
- [ ] Primary axis alignment (start/center/end/space-between)
- [ ] Counter axis alignment (start/center/end/baseline)
- [ ] Sizing mode per layer (fixed/hug/fill)
- [ ] Min/max width and height constraints
- [ ] Wrap behaviour (if enabled)

## 5. Typography details

- [ ] Font family
- [ ] Font size
- [ ] Font weight
- [ ] Line height (note format: %, px, or auto)
- [ ] Letter spacing
- [ ] Text transform (uppercase, lowercase, none)
- [ ] Text alignment (left/center/right)
- [ ] Truncation / overflow behaviour (if specified)
- [ ] Max lines (if clamp is set)

## 6. Interactive states

- [ ] Hover state (property changes from default)
- [ ] Focus state
- [ ] Active/pressed state
- [ ] Disabled state
- [ ] Loading state (if applicable)
- [ ] Error state (if applicable)

Note which states exist in Figma and which are absent — missing states should be flagged during the review.

## 7. Responsive variants

- [ ] Breakpoint-specific frames or variants (if any)
- [ ] Layout changes at different sizes
- [ ] Spacing or sizing differences across breakpoints

If the Figma source is a single fixed-size frame with no responsive annotations, note this and skip.

## 8. Nested components

- [ ] List of child components used (e.g. Icon, Badge, Avatar)
- [ ] Which variant/props are set on each child instance
- [ ] Slot structure (leading/trailing elements, content areas)

Don't extract the full internals of nested components — just their interface: what component, which variant, what props.
