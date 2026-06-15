# Translating Figma values into S2 style-macro idiom

The trap in design-to-code work is copying Figma's raw output literally: `padding:
17px`, `color: #6E6E6E`. That technically matches the mock but rots the codebase —
it bypasses the design system, breaks dark mode, and fails the repo's "no hardcoded
hex" rule. Your job is to express the *intent* of the Figma values in this repo's
Spectrum 2 idiom.

## Get the values out of Figma

Use the Figma MCP on the node the designer linked:

- `get_design_context` — structured layout/style data (the richest source: spacing,
  fills, typography, auto-layout gaps, corner radii).
- `get_variable_defs` — the design variables/tokens the node references. **This is
  gold**: if Figma already uses a named token (e.g. `gray-600`, `spacing-200`),
  that maps almost directly to an S2 token and you should prefer it over the raw
  pixel value.
- `get_screenshot` — the rendered frame, for side-by-side visual comparison.
- `get_metadata` — node names/structure when you need to find the right sub-node.

Always prefer a **named Figma variable** over a literal value — it tells you the
designer's semantic intent, which is what you want to preserve.

## Spacing & sizing: px → S2 scale

S2's style macro uses a numeric scale (roughly multiples of 4) rather than raw px.
Map Figma px to the nearest scale step:

| Figma px | S2 style value (approx) |
|----------|-------------------------|
| 4        | `4`                     |
| 8        | `8`                     |
| 12       | `12`                    |
| 16       | `16`                    |
| 24       | `24`                    |
| 32       | `32`                    |

Odd Figma values (`17px`, `15px`) almost always mean the design rounded to the
scale — snap to the nearest step (`16`) unless the designer insists on the exact
value. Confirm `get_style_macro_property_values` (react-spectrum-s2 MCP) for the
valid values of a given property when unsure.

## Color: hex → S2 color token

Never write the hex. Find the Spectrum token:

1. If `get_variable_defs` gave a named color variable, find its S2 equivalent
   (`gray-600`, `blue-900`, `accent-color-1000`, etc.).
2. Otherwise, identify the role (text, border, background, accent) and pick the
   token by role + shade. Use the react-spectrum-s2 MCP (`get_s2_page` for "Colors"
   / `get_style_macro_property_values` for `color`) to list valid tokens.
3. Verify the token resolves correctly in **both** light and dark mode — a raw hex
   would be wrong in one of them.

If no token is within a reasonable delta of the Figma color, say so explicitly and
propose the nearest token; only fall back to a custom color as a last resort, and
then check WCAG 2.2 contrast against its background.

## Typography

Figma gives `font-size / line-height / weight / family`. S2 prefers semantic font
shorthands (`font: 'body'`, `font: 'heading'`, `font: 'title'`, with size
modifiers) over loose numbers. Match the Figma text style to the closest semantic
S2 font first; reach for explicit `fontSize`/`fontWeight`/`lineHeight` only when the
mock genuinely diverges from a semantic style.

## Border radius, borders, shadows

Map to S2 tokens/scale the same way (`borderRadius`, `borderWidth`, `boxShadow`).
Prefer named/scale values over raw px.

## Sanity rule

After mapping, the resulting `style({})` should look like code a Spectrum-fluent
engineer on this team would have written by hand — semantic tokens, scale steps,
no stray hex or odd px. If it looks like a literal Figma export, redo the mapping.
