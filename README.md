<div align="center">

```
██████╗  ██████╗ ███████╗███████╗ █████╗ ██╗
██╔══██╗██╔═══██╗██╔════╝██╔════╝██╔══██╗██║
██████╔╝██║   ██║█████╗  █████╗  ███████║██║
██╔══██╗██║   ██║██╔══╝  ██╔══╝  ██╔══██║██║
██║  ██║╚██████╔╝███████╗███████╗██║  ██║██║
╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚══════╝╚═╝  ╚═╝╚═╝
```

### AI Expert · Strategist · Builder

An interactive single-page portfolio site with a **Spline 3D robot** that
follows your mouse — built as one self-contained HTML file, zero build step.

[![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white)](#)
[![Spline](https://img.shields.io/badge/Spline-3D-7c3aed)](https://spline.design)
[![No Build](https://img.shields.io/badge/build-none-brightgreen)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](#-license)

</div>

---

## ✨ Features

- 🤖 **Interactive 3D hero** — a Spline robot model that **rotates toward your
  cursor** on both axes (left/right + up/down) with smooth, weighted motion.
- 🎯 **Mouse-tracking** powered by `@splinetool/runtime` for full programmatic
  control of the scene graph.
- 🌫️ **Scroll-aware** — the 3D model stays pinned to the viewport and gracefully
  fades out as you scroll into the content below.
- 🧠 **Neural-network scroll scrubber** — a canvas animation that assembles a
  network as you scroll, with stage-based narrative text.
- 📱 **Fully responsive** — adapts from desktop to mobile with a directional
  vignette that keeps hero copy readable over the 3D.
- ⚡ **Zero dependencies to install** — a single `index.html`. The 3D runtime is
  loaded from a CDN (`esm.sh`) at runtime.

---

## 🚀 Quick Start

> **Important:** This site uses ES modules and fetches a 3D scene, so it **must
> be served over HTTP** — opening `index.html` directly with `file://` is
> blocked by browser security.

### 1. Clone

```bash
git clone https://github.com/roeea2/roeeai-spline.git
cd roeeai-spline
```

### 2. Serve locally

Pick whichever you have installed:

```bash
# Python 3
python3 -m http.server 3333

# Node.js
npx serve -l 3333

# PHP
php -S localhost:3333
```

### 3. Open

Visit **http://localhost:3333** in your browser. Move your mouse around the
hero — the robot follows. 🎉

---

## 🛠️ Usage & Customization

Everything lives in **`index.html`**. The 3D logic is in the
`<script type="module">` block near the bottom.

### Swap the 3D model

Replace the scene URL with your own Spline export
(`Spline → Export → Code → React/Viewer` reveals the `.splinecode` URL):

```js
app.load('https://prod.spline.design/YOUR_SCENE_ID/scene.splinecode')
```

### Tune the rotation

```js
tY =  nx * Math.PI * 0.5;    // horizontal range  (±90°)
tX = -ny * Math.PI * 0.18;   // vertical range    (±32°)
// ...
cY += (tY - cY) * 0.08;      // follow smoothness — lower = floatier
```

| Knob | Effect |
| --- | --- |
| `Math.PI * 0.5` | How far it turns left/right. Bigger = more dramatic. |
| `Math.PI * 0.18` | How far it tilts up/down. Kept smaller — full pitch looks unnatural. |
| `0.08` | Lerp factor. `~0.05` floaty, `~0.12` snappy. |

### Edit the content

Hero copy, services, process, testimonials, and contact are all plain HTML
sections in `index.html` — edit them directly.

---

## 📦 Tech Stack

| Layer | Choice | Why |
| --- | --- | --- |
| 3D engine | [`@splinetool/runtime`](https://www.npmjs.com/package/@splinetool/runtime) via **esm.sh** | Direct object access (the `<spline-viewer>` web component doesn't reliably expose it; esm.sh bundles deps, unpkg doesn't). |
| Rendering | `<canvas>` + `requestAnimationFrame` | Smooth per-frame lerp for the mouse follow. |
| Styling | Hand-written CSS | No framework, no build, no `node_modules`. |

---

## 📁 Project Structure

```
roeeai-spline/
├── index.html                 # The entire site (markup + styles + 3D logic)
├── superkid_robot_copy.spline # Source Spline scene file
├── skills/
│   └── spline-hero/           # Reusable Claude Code skill (see below)
└── README.md
```

---

## 🧩 Bonus: the `spline-hero` Claude Code Skill

This project's 3D integration was distilled into a reusable
[Claude Code](https://claude.com/claude-code) skill that teaches the agent how
to embed any Spline object into a hero section — including the non-obvious
gotchas (serve over HTTP, use `esm.sh`, find the right scene object, avoid the
`<spline-viewer>` pitfalls).

It lives in [`skills/spline-hero/`](skills/spline-hero):

```
skills/spline-hero/
├── SKILL.md                  # when-to-use + approach + build steps
├── assets/hero-template.html # drop-in reference implementation
└── reference.md              # symptom-first troubleshooting checklist
```

**Install it** by copying into your Claude Code skills directory:

```bash
cp -R skills/spline-hero ~/.claude/skills/        # user-wide
# or, per-project:
cp -R skills/spline-hero /path/to/project/.claude/skills/
```

Then ask Claude *"add a Spline scene to my hero"* and it handles the rest.

### Will it follow the mouse?

The template ships with the full mouse-tracking code, but it follows the mouse
**only after two things are true**:

1. **You replace the scene URL.** The template ships with a placeholder —
   nothing loads until it points at a real scene:
   ```js
   const SCENE_URL = 'https://prod.spline.design/REPLACE_ME/scene.splinecode';
   ```
2. **It grabs the right object.** The template auto-picks the scene object with
   the most children (a model's root group), which works for most scenes. But
   the *wrong* object can get picked — if you see the **background rotating**
   instead of the model, read the `console.table` it logs and hard-code the
   correct one:
   ```js
   model = app.findObjectByName('YourObjectName');
   ```

In other words: the code is there and works (this site is proof), but for an
arbitrary scene expect to confirm step 2 via the console — which is exactly why
the skill logs the scene graph and documents the fix.

---

## 📄 License

[MIT](https://opensource.org/licenses/MIT) © RoeeAI

---

<div align="center">

**Built with 🤖 and a little 3D magic.**

</div>
