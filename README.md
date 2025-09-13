# Particle Morph — Interactive 3D Text Morphing with Three.js + GSAP

**Particle Morph** is an eye-catching, high-performance demo that morphs a rotating 3D particle-sphere into user-specified text and back — built with **Three.js** (WebGL), **GSAP** for smooth animations, and a lightweight HTML/CSS UI. Type any short text and watch thousands of particles rearrange themselves into your message, then smoothly return to a sparkling sphere.

This README covers what the project does, how it works, how to run it locally, customization tips, performance optimizations, known issues & improvements, and licensing.

---

# Table of contents

* [Demo](#demo)
* [Features](#features)
* [Files / Project structure](#files--project-structure)
* [How it works (overview)](#how-it-works-overview)
* [Quick start — run locally](#quick-start--run-locally)
* [Usage](#usage)
* [Configuration & customization](#configuration--customization)
* [Performance tips](#performance-tips)
* [Troubleshooting](#troubleshooting)
* [Potential improvements](#potential-improvements)
* [Credits & license](#credits--license)

---

# Demo

> Type some text into the input and press **Create** (or Enter). The particles will morph into the text, hold briefly, then morph back to the rotating sphere.

---

# Features

* 3D particle sphere generated using `THREE.BufferGeometry` with per-vertex colors.
* Smooth morph animations between sphere and text using **GSAP**.
* Text → particles mapping is created via an offscreen canvas (pixel sampling).
* Lightweight, modern UI with responsive CSS (desktop + mobile-friendly).
* Customizable parameters: particle count, font size, colors, morph durations.
* Small dependency footprint — just Three.js + GSAP (via CDN in the example).

---

# Files / Project structure

```
/project-root
├─ index.html         ← Markup (container + input UI)
├─ styles.css         ← Styling and responsive layout
├─ script.js          ← Three.js scene, particle logic, GSAP morphs
└─ README.md          ← (this file)
```

> The code you provided uses CDN imports for Three.js and GSAP. If you want to bundle later, see the "Optional: bundling" section.

---

# How it works (overview)

1. **Scene, camera, renderer**
   `init()` creates a Three.js `Scene`, `PerspectiveCamera`, and `WebGLRenderer`. `renderer.domElement` is appended to `#container`.

2. **Particles (BufferGeometry)**
   A `BufferGeometry` is created with `count` particles → position and color attributes (`Float32Array`). Particles are displayed using `THREE.PointsMaterial` (vertexColors, additive blending, small point size).

3. **Sphere distribution**
   The project uses a spherical distribution function (similar to a spherical fibonacci-ish approach) to generate evenly spread points on a sphere and assigns colors based on depth for a nice gradient.

4. **Text sampling**
   An offscreen `<canvas>` draws the user text and then samples pixels. Bright pixels become candidate points for text. The sample is downscaled and randomly subsampled to limit points.

5. **Morphing**
   When the user submits text, `morphToText(text)`:

   * Creates the target 2D points from the canvas (`createTextPoints`).
   * Builds a `targetPositions` Float32Array: first N points map to text coordinates (Z = 0), remainder are pushed to random space.
   * Uses GSAP to animate numeric indices of the positions array toward the target values and flags `needsUpdate` on update so Three.js re-uploads attributes.

6. **Return morph**
   After a delay, `morphToCircle()` re-computes spherical positions and animates back, updating both positions and color attribute arrays.

---

# Quick start — run locally

**Prerequisites:** modern browser (Chrome, Firefox, Edge), Node.js optional.

### Option A — Open file (quick)

1. Clone / copy files to a folder.
2. Open `index.html` directly in a browser.

   * Note: some browsers restrict `file://` for modules or cross-origin behaviors — if the scene doesn't load, use a file server (Option B).

### Option B — Local static server (recommended)

From the project folder:

**Using Python (if you have Python 3):**

```bash
# macOS / Linux
python3 -m http.server 8000

# Windows (PowerShell)
python -m http.server 8000
```

Open `http://localhost:8000`.

**Using npm http-server:**

```bash
npm install -g http-server
http-server -c-1
```

### Option C — With a bundler (optional)

If you want to integrate with a build system (parcel/webpack/vite) to import NPM packages:

```bash
npm init -y
npm install three gsap
# configure bundler of your choice and import modules in script.js
```

---

# Usage

* Type up to 20 characters into the input box (this example uses `maxlength="20"`).
* Click **Create** or press **Enter**.
* The particle cloud will morph into your text, hold, and morph back into the rotating sphere.
* Resize handling is implemented — the canvas will scale with the window.

---

# Configuration & customization

Open `script.js` to modify:

* `const count = 12000;` — number of particles. Lower it for better performance on low-end devices.
* Sphere radius & jitter: change the multiplier `8` and random offsets in `sphericalDistribution`.
* Particle size: `PointsMaterial({ size: 0.08 })`.
* Font & sampling: in `createTextPoints` change `fontSize`, `ctx.font`, `threshold`, and the sampling probability `if (Math.random() < 0.3)` to control how dense the text will be.
* Morph timings & easing: GSAP calls use `duration` and `ease: "power2.inOut"` — alter to taste.
* Colors: color logic uses `color.setHSL( ... )` — adjust H/S/L ranges in both sphere and morph functions.
* UI: tweak `styles.css` for different fonts or layouts (currently uses Inter via Google Fonts).

---

# Performance tips

* **Reduce particle count** for mobile: drop `count` to 2k–6k for phones.
* **Lower point size** and disable additive blending to reduce fill-rate cost.
* **Batch attribute updates**: instead of creating thousands of GSAP tweens (one per float), a higher-performance approach is to create a single GSAP tween that interpolates a smaller set of parameters or an intermediary array and apply them to the typed array in `onUpdate`. (This avoids creating 36k separate tweens).
* **Use GPU-based morphs**: for large particle counts, consider using a `ShaderMaterial` and buffer textures + a GPU transform (ping-pong / GPGPU) to morph positions entirely on the GPU.
* **Throttle animation** on hidden tabs; browsers already throttle, but you can also pause `requestAnimationFrame` when not visible (use the Page Visibility API).

---

# Troubleshooting

**Blank screen**

* Ensure CDN scripts are reachable. If blocked (corporate networks), download and serve local copies of Three.js and GSAP.
* Check browser console for errors. Common issues:

  * `THREE is not defined` → Three.js not loaded.
  * `document.getElementById('container') is null` → HTML structure changed.

**Text looks sparse or huge**

* Adjust `fontSize` and the sampling probability in `createTextPoints`.
* Increase `count` to provide more particles available to form letters.

**Too slow on mobile**

* Lower `count`, reduce `size` or disable `AdditiveBlending` and reduce opacity.
* Remove per-particle GSAP tweens and use a single tween to drive interpolation (see Performance tips).

---

# Potential improvements (ideas)

* Replace current spherical distribution with a true Fibonacci sphere algorithm for even distribution.
* Use a single typed-array interpolation approach instead of thousands of per-index GSAP tweens (huge perf win).
* Use GPU morphing (GPGPU / transform feedback or compute shaders via WebGL2) to support 100k+ particles.
* Add color palettes and a UI color picker.
* Add camera controls (orbit/pan/zoom) and touch gestures.
* Export animation frames or record a short video.
* Accessibility: add ARIA attributes & keyboard navigation for UI.

---

# Known caveats in this code

* The example currently spawns *individual GSAP tweens per float index* to animate `position.array[i]`. Creating thousands of separate tweens may be fine for moderate `count` values but may cause overhead when `count` is very large. Consider batching or a custom interpolation loop for higher particle counts.
* Text sampling uses `Math.random() < 0.3` to reduce density. This makes results non-deterministic between runs — intentional for variety but may be undesirable if you want consistent shapes.

---

# Example snippets

**Change number of particles**

```js
// script.js
const count = 6000; // fewer particles for better mobile performance
```

**Change font used for text sampling**

```js
// script.js -> createTextPoints
ctx.font = `bold ${fontSize}px 'Inter', Arial, sans-serif`;
```

**Serve locally with Python**

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

---

# License & credits

**Author / Contributors:** You (the project owner) — add your name and GitHub link here.

**Libraries used**

* \[Three.js] — WebGL 3D library
* \[GSAP] — animation engine

**License:** MIT (suggested)

```
MIT License

Copyright (c) YEAR Your Name

Permission is hereby granted...
```

*(Replace `YEAR` and `Your Name` accordingly.)*

---

# Final notes

This project is an excellent base for interactive hero sections, creative landing pages, or procedural visuals. If you'd like, I can:

* Turn this README into a polished GitHub README with badges and a GIF example.
* Suggest or implement a performance-optimized version (single-tween interpolation or GPU morph).
* Add deploy instructions to GitHub Pages and a minimal `package.json` and build setup.

Which of those would you like next?#sphere_particle
