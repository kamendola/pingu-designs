# Isolation workbench — one component, centered, blank canvas

When the designer wants to perfect a single thing, the rest of the app is noise:
data loads, layout shifts, and — in most real apps — an auth/login wall. The
workbench renders just the target component, centered on an empty page, inside all
the context it needs (S2 `Provider`, plus whatever the app wraps with — Intl, a
user/session provider, a query client), but **outside** the auth guard.

The trick is generic to React Router apps: most apps mount one or two *public*
routes (a sign-out page, an error page, a public landing) as siblings of the
protected route tree. Anything you add at that level renders without
authentication. You add one such route, use it, then remove it.

Everything here is **temporary scaffolding**. The teardown step is not optional —
the app must end exactly as it started, plus your real style edits.

## Setup

### 1. Find where public routes are declared

Look at the app's router (often `App.tsx`/`routes.tsx`). Identify the public routes
that sit *outside* the auth-guarded `/*` (or equivalent) route — e.g. a `/signout`
or `/error` route. That sibling level is where your sandbox route goes.

If the app has no auth wall, this is even simpler — any route works.

### 2. Create the sandbox page

A scratch page (a leading underscore in the filename, e.g. `_DesignSandbox.tsx`,
signals "temporary"). Use the project's own license/copyright header with the
current year. Import the target component, center it, pass minimal mock props.
Pick a unique sentinel string to tag every temporary edit — below it's
`DESIGN-SANDBOX`:

```tsx
/* <project copyright header, current year> — TEMPORARY design sandbox, delete before commit. */
import { style } from '@react-spectrum/s2/style' with { type: 'macro' }
import TargetComponent from '@/components/.../TargetComponent'

// DESIGN-SANDBOX (temporary)
export default function DesignSandbox() {
  return (
    <div
      className={style({
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        minHeight: 'screen',
        width: 'full'
      })}
    >
      <TargetComponent /* mock the minimal props it needs to render */ />
    </div>
  )
}
```

If the component reads from a user/session hook, a data-fetching client, or other
context and crashes unauthenticated, give it mock props or wrap it in a minimal
local context value — just enough to render. Keep mocks in this file only.

### 3. Register a public route

Add it as a sibling of the existing public route(s), **above/outside** the
protected route tree, tagged with the sentinel comment so teardown is unambiguous:

```tsx
{/* DESIGN-SANDBOX (temporary) */}
<Route path="/design-sandbox" element={<DesignSandbox />} />
```

…plus the matching import (also sentinel-tagged). These router edits are the only
structural changes this skill permits, and they exist solely to be deleted.

### 4. View it

Start the dev server the way the project documents, and open the sandbox path
(e.g. `/design-sandbox`). No login is needed for a public route. HMR means your
style edits to the real component show up live as you iterate.

## States gallery mode (every variant + state at once)

Designs usually specify a *matrix* of states — variants, sizes, selected/disabled,
hover/focus/press, light/dark. Instead of a single centered instance, render the
component **many times in a labeled grid** so you can compare every state against
the Figma frame side by side. This is the workbench's most useful mode for a polish
pass, and it all lives in the throwaway sandbox file, so it breaks no rules.

Keep it **data-driven** (per the skill's rules): declare the axes as arrays and map
over them through one generic cell template — never hand-write each combination.

```tsx
// DESIGN-SANDBOX (temporary)
import { style } from '@react-spectrum/s2/style' with { type: 'macro' }
import { Button } from '@react-spectrum/s2'

// Axes of variation — edit to match what the design enumerates.
const variants = ['primary', 'secondary', 'accent', 'negative'] as const
const sizes = ['S', 'M', 'L', 'XL'] as const
const flags = [
  { label: 'default', props: {} },
  { label: 'disabled', props: { isDisabled: true } }
]

const grid = style({ display: 'grid', gap: 24, padding: 32, justifyItems: 'start' })
const cell = style({ display: 'flex', flexDirection: 'column', gap: 8 })
const caption = style({ font: 'detail-sm', color: 'gray-600' })

function Gallery() {
  return (
    <div className={grid}>
      {variants.flatMap(variant =>
        sizes.flatMap(size =>
          flags.map(({ label, props }) => (
            <div key={`${variant}-${size}-${label}`} className={cell}>
              <span className={caption}>{`${variant} · ${size} · ${label}`}</span>
              <Button variant={variant} size={size} {...props}>Label</Button>
            </div>
          ))
        )
      )}
    </div>
  )
}
```

### Prop-driven states — render them directly

Anything the component exposes as a prop is shown statically by just passing it:
`variant`, `size`, `isDisabled`, `isSelected`, `isQuiet`, `density`, etc. These are
the bulk of most designs and need no interaction to see.

### Interaction states (hover / focus / press) — three ways

These are CSS/render-prop states you can't set with a plain prop, so:

1. **Live** (default): the grid cells are real components — hover, tab to, and press
   them in the browser to see each state. Best for quick iteration.
2. **Freeze for comparison:** in DevTools, the Styles panel **`:hov`** toggle forces
   `:hover` / `:focus` / `:focus-visible` / `:active` on the selected element so the
   state holds still while you compare to Figma. If driving the browser via
   automation (only on an explicit "show me"), force the pseudo-state, then capture.
3. **Custom components you own:** if the target is a custom RAC + style-macro
   component whose `className` is a render-prop function (`className={rp =>
   styles({...rp})}`), you can mount extra cells that call that style function with
   a forced state object — e.g. `styles({ isHovered: true })` — to render the hover/
   pressed look statically. Only works for components you control the styling of;
   don't fake state on the real instance the app ships.

### Light + dark together

Render the gallery twice and force the color scheme per copy so you catch
`light-dark()` / token issues in one glance. Wrap each copy in its own S2 `Provider`
with `colorScheme="light"` and `colorScheme="dark"` (or use a container that spreads
`setColorScheme()`), and give the dark side a dark background so it reads correctly.

## Iterate

Edit styles on the **real component** (not the sandbox wrapper or the gallery
scaffolding). The sandbox only provides the stage. Compare each cell against the
matching Figma state (`get_screenshot`) until they match.

## Teardown (mandatory)

1. Delete the sandbox page file.
2. Remove the two sentinel-tagged lines from the router (route + import).
3. Confirm nothing temporary remains — grep for your sentinel and the sandbox file;
   both should come back empty:
   ```bash
   git grep -n "DESIGN-SANDBOX"
   ```
4. `git diff` should now show **only** the intended style changes to the real
   component — no sandbox file, no router changes.
5. Re-run the type-check (and build, if quick) after teardown to confirm the app
   compiles in its restored state.
