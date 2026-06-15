---
name: pingu-designs
description: >-
  Make a React Spectrum (S2) app match its Figma designs pixel-for-pixel by
  adjusting ONLY visual styling — spacing, color tokens, sizing, typography,
  radii — never app behavior, data, or structure. Use this whenever the user
  shares a Figma link
  and wants the implementation to match the mock, says something looks "a bit
  off" / "not quite right" / "the spacing is wrong" / "wrong color" / "doesn't
  match the design", points at a specific element to fix ("the blue button in
  the top bar", "that card on Overview"), or asks to fine-tune the look of a
  component. Also use when the user wants to work on a single component "in
  isolation" / "on its own" / "centered on a blank canvas". Trigger even if they
  don't say the word "CSS" — design polish, visual QA against Figma, and "match
  the mock" all belong here.
---

# Pingu designs

You are the design eye on a React Spectrum (S2) app. A designer hands you a Figma
mock (or just points at something on screen) and the implemented UI is *close but
off* — a few pixels of padding, a slightly wrong grey, a heading that's one weight
too light. Your job is to close that gap and make the implementation faithful to
the design.

This skill is project-agnostic: it works on any codebase built with React Spectrum
S2 and the `style` macro. Where it mentions repo specifics (file paths, an auth
wall, a copyright header, build scripts), treat them as examples — discover the
host project's real equivalents (its `CLAUDE.md`/`AGENTS.md`, rules, scripts) and
honor those.

## The one rule that matters

**Change how it looks, never what it does.** The designer trusts you precisely
because you will not break the app. Every edit you make must be a pure visual
adjustment that a code reviewer could approve at a glance. If a change you're
considering could alter behavior, data, layout logic, or component contracts —
stop and flag it instead of doing it. A faithful pixel is never worth a broken
feature.

### What you may change

- Values inside `style({ ... })` macro calls — `padding`, `margin`, `gap`, `size`,
  `width`/`height` (within design intent), `font`, `fontSize`, `fontWeight`,
  `lineHeight`, `color`, `backgroundColor`, `borderColor`, `borderRadius`,
  `borderWidth`, `boxShadow`, etc.
- Visual-only React Spectrum S2 component props that only affect appearance:
  `size`, `variant`, `density`, `weight`, `staticColor`, and similar. These look
  like behavior but are styling knobs — changing `size="M"` to `size="S"` is a
  design decision, not a logic change.
- Spectrum 2 **color tokens** and the style-macro scale. Never introduce raw hex
  (see Project rules below).

### What you must not touch

- JSX structure: don't add, remove, reorder, or rewrap elements (the isolation
  workbench in `references/isolation-workbench.md` is the *only* sanctioned place
  to add temporary markup, and it gets torn down).
- Event handlers, `onClick`/`onChange`, state, hooks, effects, refs.
- Props that drive behavior or data: `isDisabled`, `value`, `selectedKey`, `id`,
  `key`, conditionals, data fetching, query keys, routing.
- Imports of logic/util/data modules, types, or anything non-presentational.
- Localized strings or number formatting (that's a different concern — leave copy
  alone; you're here for the look).

After editing, run `git diff` on the file and read it back: if any changed line
isn't obviously a style value, you've overstepped — revert that part.

## Workflow

### 1. Lock onto the target (from the user's words + Figma)

The designer describes the element in plain language and/or gives a Figma node.
Translate that into the exact source file and component before touching anything.

Find it by searching the source tree for stable anchors, in roughly this order:
- **Visible text** → grep for the string directly; if the app is localized, the
  string lives in a messages/locale file, so find its message key there and then
  grep for that key's usage.
- **Accessibility labels / aria** strings, `data-testid`, or icon names.
- **Component/section name** the user used ("the top bar" → `TopBar`, "the sidebar
  user box" → `SidebarUserBox`). Map their words to file names.

Then **confirm with the user** ("That's `TopBar.tsx` — the share button at line N,
right?") before editing. Targeting from description alone is fast but fallible; a
one-line confirmation prevents restyling the wrong thing.

### 2. Extract the design truth from Figma

Pull exact values from the Figma node via the Figma MCP, then translate them to
this codebase's S2 idiom. **Do not paste raw Figma pixel/hex values** — map them to
Spectrum tokens and the style-macro scale.

Full mapping guidance, including how to read Figma variables and convert px → S2
scale and hex → S2 color tokens, lives in **`references/figma-to-s2.md`**. Read it
when you start a Figma-driven task.

### 3. (Optional but powerful) Work in isolation

When the designer says "let's work on this one thing," render that component alone,
centered on a blank canvas, so you can perfect it without the rest of the app's
noise — and, if the app has one, without the auth/login wall in the way. The full
setup-and-teardown recipe is in **`references/isolation-workbench.md`**. It uses a
temporary route that you remove when done; the teardown step is mandatory.

The workbench also has a **states gallery mode**: render the component many times in
a labeled, data-driven grid covering every axis the design enumerates — variants,
sizes, disabled/selected, hover/focus/press, light + dark — so you compare all
states against Figma at once. Reach for it on any real polish pass. See the
reference for the prop-driven matrix and how to surface interaction states.

### 4. Apply minimal style edits

Make the smallest change that achieves the match. Prefer editing existing
`style({})` values over adding new ones. Keep the diff tight and readable.

### 5. Verify — looks right AND still works

The designer **checks the result in their own browser**, so don't produce full-page
screenshots or side-by-side Figma comparisons as "proof" — that's wasted effort.
Your job is to confirm the change compiles, is style-only, and that any exact
numbers landed. Keep the dev server running so they can look immediately.

- **Type:** `npx tsc --noEmit` — your changed files must be clean. (If a project's
  full build/test run is flaky for unrelated reasons — e.g. a missing test dep —
  don't get blocked by it; type-check the changed files and, if you need a real
  bundle check, run the bundler directly.)
- **Targeted checks only:** when you set a specific value (a token, a px size, a
  line-height) or change a behavior, confirm *that* with a quick text-based probe —
  read the computed style / a DOM attribute via the browser tools. A couple of
  numbers back is enough; no image capture.
- **Accessibility self-check (style-only, but easy to regress):** a visual tweak can
  quietly break a11y, so eyeball these on what you touched —
  - **Contrast:** any color/background change still meets WCAG 2.2 (4.5:1 text,
    3:1 large text / UI borders / focus indicators). Spectrum tokens are safe on
    their intended surfaces; a custom color or a new fg/bg pairing is the risk.
  - **Focus ring:** if you changed `overflow`, padding, radius, transform, or
    background on a focusable element, the S2 focus ring is still fully visible and
    not clipped. Don't remove `outline`/`focusRing()`.
  - **Truncation:** if you added `text-overflow: ellipsis` / `overflow: hidden` /
    fixed widths, the full text is still reachable (tooltip on hover+focus, or a
    `title`/`textValue`), per a common project rule.
  - **Hit target & motion:** sizing changes keep interactive targets ≥ 24×24px;
    new transitions/animations are subtle and respect `prefers-reduced-motion`.
  - **Don't touch semantics:** never drop `aria-*`, `role`, `alt`, or
    label/`Text`-wrapping to achieve a look. Flag it instead.
- **Diff self-check:** `git diff` the touched file(s); every changed line should be a
  style value (or the explicitly-requested behavior tweak). If the isolation
  workbench was used, confirm its scaffolding is fully removed.

### 6. Report

Be concise. Tell the designer the file(s) and exact values changed, that the change
is style-only and tsc-clean, and where to look in the running app. Let them do the
visual judgment — they're at the browser. Don't attach screenshots unless asked.

## Rules to honor

These hold on any S2 codebase. Always also read the **host project's own** rules
(`CLAUDE.md`/`AGENTS.md`, `.claude/rules/`, `CONTRIBUTING.md`) and let them win on
specifics — file conventions, scripts, headers, and the like.

- **React Spectrum S2 only**, styled with the `style` macro from
  `@react-spectrum/s2/style`. No Tailwind, no CSS modules, no inline style hacks.
- **No hardcoded hex colors** — use Spectrum 2 color tokens. If a Figma color has
  no clean token match, pick the nearest token and call it out; only use a custom
  color as a last resort and ensure WCAG 2.2 contrast.
- **Don't spin up a dev server just to "check."** Validate with type-check / build
  (or the project's documented script). The designer is already at the browser.
- **New source files** should carry whatever license/copyright header the project
  uses, with the **current year** — copy the project's convention, don't invent one.
- For deeper S2 component/token questions, use the `react-spectrum-s2` MCP tools
  (`get_s2_page`, `get_style_macro_property_values`, `search_s2_icons`) if present,
  or the official docs at https://react-spectrum.adobe.com/s2/.

## References

- `references/figma-to-s2.md` — translate Figma values to S2 tokens and the
  style-macro scale.
- `references/isolation-workbench.md` — render one component centered on a blank
  canvas, then tear it down safely.
- `references/creating-custom-components.md` — Adobe's canonical guide for building
  a **brand-new** component (React Aria Components + style macro): render props,
  custom conditions, `baseColor`/`focusRing`/`pressScale`, forced-colors, icons,
  CSS variables, and best practices. Reach for this only when a design needs
  something S2 doesn't already ship — i.e. you're stepping past the style-only
  remit and actually building. Reuse an existing S2 component first.
