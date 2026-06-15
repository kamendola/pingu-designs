![hero](https://raw.githubusercontent.com/kamendola/pingu-designs/refs/heads/main/hero.png)

# Pingu designs

A [Claude Code](https://docs.claude.com/en/docs/claude-code) **skill** for design-to-code polish on any [React Spectrum S2](https://react-spectrum.adobe.com/s2/) app.

Hand it a Figma link (or just point at something on screen) and it makes the implementation faithful to the design by changing **only visual styling** — spacing, color tokens, sizing, typography, radii — **never** behavior, data, or structure.

## What it does

- **Targets elements from plain language** — "the blue button in the top bar" → finds the source file, confirms, then edits.
- **Maps Figma → S2 the right way** — converts px to the style-macro scale and hex to Spectrum color tokens (never raw values). See [`references/figma-to-s2.md`](references/figma-to-s2.md).
- **Isolation workbench** — renders a single component on a blank canvas, including a **states gallery** (variants × sizes × disabled/selected × hover/focus/press × light/dark). See [`references/isolation-workbench.md`](references/isolation-workbench.md).
- **Builds new components correctly** when needed — React Aria + style macro. See [`references/creating-custom-components.md`](references/creating-custom-components.md).
- **Accessibility self-check** baked into verification — WCAG contrast, focus-ring visibility, truncation tooltips, hit-target size, never dropping ARIA/semantics.

## Install

Clone into your personal Claude skills folder:

```bash
git clone https://github.com/kamendola/pingu-designs.git ~/.claude/skills/pingu-designs
```

Or, for a single project, into that repo's `.claude/skills/` instead.

Restart Claude Code (or reload the app) and invoke it with `/pingu-designs`, or just share a Figma link and ask to match the design.

## Notes

- Generic to any React Spectrum S2 codebase — it reads the host project's own rules (`CLAUDE.md`/`AGENTS.md`, `.claude/rules/`) and lets them win on specifics.
- `references/creating-custom-components.md` is adapted from Adobe's open-source (Apache-2.0) React Spectrum agent skill ([PR #9939](https://github.com/adobe/react-spectrum/pull/9939)).
