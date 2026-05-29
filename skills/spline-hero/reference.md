# Spline Hero — Troubleshooting

Symptom-first checklist. Each row is a real failure seen building this.

## Nothing renders / blank canvas

- **Opened via `file://`.** Console shows `Unsafe attempt to load URL … 'file:'
  URLs are treated as unique security origins`. Fix: serve over HTTP —
  `python3 -m http.server 3333`, open `http://localhost:3333`.
- **Wrong import host.** `@splinetool/runtime` from unpkg loads but is broken
  (unbundled deps). Use `https://esm.sh/@splinetool/runtime`.
- **Bad scene URL.** Must be `prod.spline.design/<id>/scene.splinecode`, not the
  `my.spline.design/<slug>/` share URL. Check the console for a load error.

## Scene loads but the model won't rotate

- **No object reference.** Look for `[spline-hero] ✓ rotating: <name>` in the
  console. If absent, the load promise rejected — read the error.
- **Grabbed the wrong object** (camera, light, or background rotates / nothing
  visible moves). Read the logged `console.table`, find the model's root group
  (usually the row with the most children), and hard-code:
  `model = app.findObjectByName('ExactName')`.
- **Scene animation overrides rotation.** If the scene has a looping animation on
  that object, it fights your `rotation.y`. Target a parent/empty wrapper, or a
  child that isn't animated.

## The whole page tilts instead of the model

You applied a CSS `transform: rotateY()` to the canvas/viewer element. Remove it.
Rotation must be set on the **Spline object** (`model.rotation.y`), not the DOM
node.

## Scrolling zooms the 3D scene instead of scrolling the page

Spline's camera captures the wheel event. Either set `pointer-events: none` on
the canvas container (simplest — also stops drag-orbit), or add a hero `wheel`
listener that calls `e.stopPropagation()`.

## `<spline-viewer>` web component issues

If you (or the user's existing code) use `<spline-viewer>`:
- `viewer.application` is frequently `undefined`.
- The `load` event's `detail` payload varies by version.
Migrate to `@splinetool/runtime` + `<canvas>` (see `assets/hero-template.html`)
for reliable object access.

## Model scrolls off-screen / overlaps sections below

Use `position: fixed` on the container and fade its opacity to 0 across the first
viewport of scroll, then `visibility: hidden`. See the scroll-fade script in the
template.

## "Your .splinecode file is more recent than the library"

The pinned runtime version is older than the scene. Remove the version pin so
esm.sh serves the current build: `https://esm.sh/@splinetool/runtime`.

## Tuning the feel

- Rotation range: `Math.PI * 0.5` ≈ ±90° (Y), `Math.PI * 0.18` ≈ ±32° (X).
- Follow smoothness: the `0.08` lerp factor. Lower = floatier/laggier, higher =
  snappier. ~0.05–0.12 is a good band.
- Vertical feels best smaller than horizontal — a full pitch looks unnatural.
