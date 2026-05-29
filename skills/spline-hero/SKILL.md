---
name: spline-hero
description: >-
  Embed an interactive Spline 3D object (robot, character, product, etc.) as a
  full-screen hero background that rotates to follow the mouse and fades on
  scroll. Use when the user wants to add a Spline scene to a landing/hero
  section, make a Spline model track or look at the cursor, or fix a Spline
  embed that won't load, won't expose its objects, or scroll-zooms.
---

# Spline Hero

Build a full-screen hero section with an interactive Spline 3D scene whose model
rotates to follow the mouse (left/right + up/down) and fades out as the user
scrolls past the hero.

## What you need from the user

1. **Scene URL.** Either:
   - A `prod.spline.design/<id>/scene.splinecode` URL (preferred — direct runtime URL), or
   - A `my.spline.design/<slug>/` share URL, or a React snippet containing the
     `scene="https://prod.spline.design/.../scene.splinecode"` string.

   The `.splinecode` URL is what the runtime loads. If the user only gives a
   `my.spline.design` share link, the React/export panel in Spline reveals the
   matching `prod.spline.design/.../scene.splinecode` URL — ask for it or for
   the export snippet.

## The approach that works (and the ones that don't)

Use **`@splinetool/runtime` loaded from `esm.sh`**, rendering into a plain
`<canvas>`. This is the only approach that reliably gives programmatic access to
scene objects so you can drive rotation.

Do **NOT** use these — they fail in predictable ways:

| Approach | Why it fails |
| --- | --- |
| `<spline-viewer>` web component | `viewer.application` is often `undefined`; the `load` event's `detail` is inconsistent across versions, so you can't reliably grab objects. |
| `@splinetool/runtime` from **unpkg** | unpkg does not bundle the package's dependencies for the browser; the import resolves but the module is broken. **esm.sh bundles deps** — use it. |
| `<iframe>` of the share URL | Zero programmatic control; can't rotate objects or read the scene graph. |
| CSS `transform: rotateY()` on the canvas/viewer | Rotates the whole HTML element (the entire scene tilts), not the 3D model. Looks like the page is rotating. |

## Critical requirements

- **Serve over HTTP, not `file://`.** ES module imports and the scene fetch are
  blocked by `file:` unique-origin security ("Unsafe attempt to load URL …").
  Always run a local server: `python3 -m http.server 3333` then open
  `http://localhost:3333`.
- **Match runtime to scene version.** Import `@splinetool/runtime` with no
  version pin (`https://esm.sh/@splinetool/runtime`) so it stays current with
  the scene. A pinned old version logs "Your .splinecode file is more recent
  than the library."

## Build steps

1. Copy `assets/hero-template.html` into the project as the page (or merge its
   hero markup, the `<style>` block, and the two `<script>` blocks into an
   existing page).
2. Replace the scene URL placeholder with the user's `.splinecode` URL.
3. Start a local server and open it over `http://localhost:...` — **never
   double-click the file**.
4. Open DevTools console. The script logs the scene's object graph as a table
   plus `[spline-hero] ✓ rotating: <name>`. Confirm it picked the model, not a
   camera/light/background.

## Finding the right object to rotate

There is no universal object name. The template:

1. Calls `app.getAllObjects()` and logs them (name, type, child count, parent).
2. Filters out cameras, lights, and background/env objects by name regex.
3. Sorts the rest by child count and picks the one with the most children — the
   model's root group contains all its parts, so it has the most children.

If it grabs the wrong object, read the logged table and hard-code the correct
name via `app.findObjectByName('<Name>')`. Ask the user to paste the table if
you can't see it.

## Mouse-tracking rotation

- Mouse **X** → `rotation.y` (turn left/right), range ≈ ±90° (`Math.PI * 0.5`).
- Mouse **Y** → `rotation.x` (look up/down), range ≈ ±32° (`Math.PI * 0.18`).
  Negate the Y term so mouse-up = look-up.
- Smooth every frame with a lerp: `current += (target - current) * 0.08`. Lower
  = floatier, higher = snappier.
- Reset targets to 0 on `mouseleave` so the model returns to rest.
- Listen on `window` (not the canvas) so movement is caught even over overlaid
  text.

## Layout & polish

- Hero is `100vh`; model canvas sits in a `position: fixed` container at
  `z-index: 0` so it stays pinned while the hero scrolls.
- Fade the container's opacity from 1→0 across the first ~85% of viewport
  scroll, and set `visibility: hidden` once invisible so it never overlaps
  sections below. `pointer-events: none` on the container keeps it from
  blocking clicks.
- Add a directional vignette gradient (dark on the text side → transparent on
  the model side) so hero copy stays readable without hiding the model.
- Stop the wheel event from reaching Spline (or use `pointer-events: none`) so
  scrolling the page doesn't zoom the 3D camera.

See `reference.md` for the full troubleshooting checklist.
