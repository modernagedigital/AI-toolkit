# Figma Constraint to CSS Mapping

Reference for translating Figma layout properties to their CSS equivalents. Use during the spacing and sizing audit phase.

## Sizing modes

| Figma sizing | CSS equivalent | Notes |
|---|---|---|
| **Fixed** (explicit px) | `width: Npx` / `height: Npx` | Direct pixel value. Check for responsive concerns — fixed sizes often need media query overrides. |
| **Hug contents** | `width: fit-content` / `width: auto` / `flex: 0 0 auto` | Element shrinks to its content. In flex contexts, `flex-shrink: 0` or `flex: none` also works. |
| **Fill container** | `width: 100%` / `flex: 1` / `flex-grow: 1` | Element stretches to fill parent. In flex: `flex: 1 1 0%`. In grid: `1fr` or spanning columns. |

## Auto-layout → Flexbox

| Figma auto-layout property | CSS flexbox equivalent |
|---|---|
| **Direction: Horizontal** | `flex-direction: row` |
| **Direction: Vertical** | `flex-direction: column` |
| **Gap** (item spacing) | `gap: Npx` |
| **Padding** | `padding` (Figma supports independent per-side values) |
| **Primary axis alignment: Start** | `justify-content: flex-start` |
| **Primary axis alignment: Center** | `justify-content: center` |
| **Primary axis alignment: End** | `justify-content: flex-end` |
| **Primary axis alignment: Space between** | `justify-content: space-between` |
| **Counter axis alignment: Start** | `align-items: flex-start` |
| **Counter axis alignment: Center** | `align-items: center` |
| **Counter axis alignment: End** | `align-items: flex-end` |
| **Counter axis alignment: Baseline** | `align-items: baseline` |
| **Wrap** | `flex-wrap: wrap` |

## Individual child overrides

| Figma child property | CSS equivalent |
|---|---|
| **Align self** (override counter axis) | `align-self: flex-start / center / flex-end / stretch` |
| **Absolute position** (child breaks out of auto-layout) | `position: absolute` with explicit `top/right/bottom/left` |
| **Fixed size on child in fill parent** | Child has `flex: 0 0 Npx` while siblings use `flex: 1` |

## Constraints (non-auto-layout frames)

For frames that don't use auto-layout, Figma uses constraint-based positioning:

| Figma constraint | CSS equivalent |
|---|---|
| **Left** | `position: absolute; left: Npx` |
| **Right** | `position: absolute; right: Npx` |
| **Left and Right** | `position: absolute; left: Npx; right: Npx` (stretches) |
| **Center (horizontal)** | `position: absolute; left: 50%; transform: translateX(-50%)` |
| **Top** | `position: absolute; top: Npx` |
| **Bottom** | `position: absolute; bottom: Npx` |
| **Top and Bottom** | `position: absolute; top: Npx; bottom: Npx` (stretches) |
| **Center (vertical)** | `position: absolute; top: 50%; transform: translateY(-50%)` |
| **Scale** | Percentage-based sizing (`width: N%`) |

## Min/Max constraints

| Figma property | CSS equivalent |
|---|---|
| **Min width** | `min-width: Npx` |
| **Max width** | `max-width: Npx` |
| **Min height** | `min-height: Npx` |
| **Max height** | `max-height: Npx` |

These are especially important for responsive components — a Figma component with min-width and fill-container should map to `flex: 1; min-width: Npx` in CSS.

## Common pitfalls

- **Gap vs margin:** Figma auto-layout uses `gap` for spacing between items. Code that applies margin to children instead of gap on the parent is functionally similar but harder to maintain and will break with reordering.
- **Padding asymmetry:** Figma supports per-side padding. Check that the code doesn't use shorthand (`padding: 16px`) when the design has asymmetric values (`padding: 16px 24px`).
- **Nested auto-layouts:** Figma commonly nests auto-layout frames. Each should map to its own flex container in CSS — collapsing them into a single element loses the layout semantics.
