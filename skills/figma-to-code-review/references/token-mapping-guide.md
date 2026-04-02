# Token Mapping Guide

Common translation patterns between Figma token names and code implementations. Use this to verify that the code references the correct token — not just that the resolved value looks right.

## Colour tokens

| Figma token | CSS Custom Property | Tailwind | Styled-components / Theme UI |
|---|---|---|---|
| `color/primary/500` | `var(--color-primary-500)` | `text-primary-500`, `bg-primary-500` | `theme.colors.primary[500]` |
| `color/neutral/100` | `var(--color-neutral-100)` | `text-neutral-100`, `bg-neutral-100` | `theme.colors.neutral[100]` |
| `color/semantic/error` | `var(--color-error)` | `text-error`, `bg-error` | `theme.colors.error` |
| `color/semantic/success` | `var(--color-success)` | `text-success`, `bg-success` | `theme.colors.success` |

**Watch for:** Figma often uses `/` separators (`color/primary/500`) while code uses `-` or `.` (`color-primary-500`, `colors.primary.500`). The structure should match even if the delimiter differs.

## Spacing tokens

| Figma token | CSS Custom Property | Tailwind | Styled-components / Theme UI |
|---|---|---|---|
| `spacing/xs` (4px) | `var(--spacing-xs)` | `p-1`, `gap-1`, `m-1` | `theme.space.xs` |
| `spacing/sm` (8px) | `var(--spacing-sm)` | `p-2`, `gap-2`, `m-2` | `theme.space.sm` |
| `spacing/md` (16px) | `var(--spacing-md)` | `p-4`, `gap-4`, `m-4` | `theme.space.md` |
| `spacing/lg` (24px) | `var(--spacing-lg)` | `p-6`, `gap-6`, `m-6` | `theme.space.lg` |
| `spacing/xl` (32px) | `var(--spacing-xl)` | `p-8`, `gap-8`, `m-8` | `theme.space.xl` |

**Watch for:** Tailwind's numeric scale (1 = 4px, 2 = 8px, etc.) is an abstraction over the pixel value. Verify the project's `tailwind.config` hasn't overridden the default scale. Also check whether the project extends the default spacing with semantic names (e.g. `spacing: { section: '3rem' }`).

## Typography tokens

| Figma token | CSS Custom Property | Tailwind | Styled-components / Theme UI |
|---|---|---|---|
| `font/family/sans` | `var(--font-sans)` | `font-sans` | `theme.fonts.sans` |
| `font/size/sm` | `var(--font-size-sm)` | `text-sm` | `theme.fontSizes.sm` |
| `font/size/base` | `var(--font-size-base)` | `text-base` | `theme.fontSizes.base` |
| `font/weight/medium` | `var(--font-weight-medium)` | `font-medium` | `theme.fontWeights.medium` |
| `font/lineHeight/normal` | `var(--line-height-normal)` | `leading-normal` | `theme.lineHeights.normal` |

**Watch for:** Figma expresses line height as a percentage (e.g. 150%) or absolute value (e.g. 24px). CSS can use unitless (1.5), percentage, or px. Verify the resolved value matches, not just the format.

## Border and shadow tokens

| Figma token | CSS Custom Property | Tailwind | Styled-components / Theme UI |
|---|---|---|---|
| `radius/sm` | `var(--radius-sm)` | `rounded-sm` | `theme.radii.sm` |
| `radius/md` | `var(--radius-md)` | `rounded-md` | `theme.radii.md` |
| `radius/full` | `var(--radius-full)` | `rounded-full` | `theme.radii.full` |
| `shadow/sm` | `var(--shadow-sm)` | `shadow-sm` | `theme.shadows.sm` |
| `shadow/md` | `var(--shadow-md)` | `shadow-md` | `theme.shadows.md` |

## Checking the full chain

In projects with abstraction layers, verify the complete mapping:

```
Figma token name
  → Design system config (e.g. tokens.json, tailwind.config.js, theme.ts)
    → Code reference (utility class, CSS variable, theme accessor)
      → Resolved browser value
```

A value can render correctly in the browser but still be wrong if it references the wrong intermediate token. For example, `bg-blue-500` might visually match `color/primary/500` today, but if the project's primary colour changes, `bg-blue-500` won't update with it.
