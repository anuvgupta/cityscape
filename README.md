# Procedural 3D Cityscape in WebGL — Notes

**Date:** May 15, 2026

## Question

Is it possible to procedurally generate a 3D cityscape with nice shaders for sunlight and maybe nice grass with wind as well, in a web browser (WebGL)? And have it look good and run on iPad browser?

## Feasibility Assessment

Yes, all of that is very doable in WebGL, and iPad Safari handles it well as long as you budget for mobile GPU limits. **Three.js** is the practical choice — it gives you WebGL2, instanced meshes, post-processing, and a sky/sun shader (the built-in `Sky` shader does Preetham atmospheric scattering and looks great with a directional light driving shadows).

### City

Instanced boxes with procedural facade textures (or shader-generated window patterns) let you render thousands of buildings in one draw call.

### Grass

The standard technique is instanced blade geometry where a vertex shader bends each blade using sampled noise + time. Three.js demos like "grass" by Mr.doob and several Shadertoy ports run this pattern.

### Sunlight & Shadows

Cascaded shadow maps or a single shadow map with a tight frustum around the camera give you nice sunlight contact shadows.

## iPad-Specific Guidelines

- Stick to **WebGL2** (supported since iOS 15)
- Keep canvas at `devicePixelRatio` capped around **1.5**
- Use a **single shadow map** at 1024–2048px
- **Instance everything**
- Avoid heavy post-processing (one bloom pass is fine, SSAO usually isn't)
- Aim for **~30–50k instances** total
- Expect **60fps on M-series iPad**, 30fps on older ones

## Prototyping Setup

Single HTML file, Three.js via CDN import map, all code in one `<script type="module">` block. No build step, no npm — just save and refresh.

## Current State (v0.13)

### Rendering / Scene

- Three.js (r166) loaded from CDN, single-file ES module
- WebGL renderer, sRGB output, `devicePixelRatio` capped at 1.5
- Brighter `#de5b68` sky with a darker `#873a42` fog (220 → 500 units) — pushed back so the city stays clear and distant buildings read as silhouettes against the sky — **v0.13**
- Ambient + directional lighting (no shadows yet)
- Grid helper + axes helper for reference
- Camera starts further back at `(130, 90, 130)`; `MAX_DISTANCE` bumped to 400 — **v0.11**

### Procedural City

- Grid oversized to ~2.2× target buildings so some lots stay empty and multi-cell footprints fit — **v0.8**
- Per-row / per-column road widths jittered around the base 3-unit spacing (`ROAD ± 1.6`) — **v0.8**
- Variable footprints: ~55% 1×1, ~15% 2×1, ~15% 1×2, ~15% 2×2, so some buildings are squares, some rectangles, some big blocks — **v0.8**
- Random cell-fill fraction per building (small lots 55–90%, larger lots 78–96%)
- Random heights bucketed into three tiers — skyscrapers (25–40), medium (8–15), small (3–7) — with distribution controlled by sliders — **v0.7**
- All buildings share a single `MeshStandardMaterial` tinted `#ed2651` — **v0.9**
- "Generate" button reseeds the layout

### Generation Settings (v0.7)

- Collapsible panel (chevron toggle), defaults collapsed so only Generate + chevron are visible
- **Buildings** slider: total count 1–500 (default 175 — **v0.10**)
- **Skyscrapers** / **Medium** sliders: percentages, mutually clamped so their sum ≤ 100 (defaults 35% / 50%, leaving 15% small — **v0.10**)
- **Small** slider: disabled, auto-updates to `100 − skyscrapers − medium`
- Expanded panel `max-height` bumped to 600px so all four sliders fit — **v0.10**
- Scene only regenerates when Generate is pressed; sliders just update labels

### Camera Controls

Two parallel input paths feeding the same camera math (`orbit`, `zoom`, `pan`, `elevate`):

**On-screen pad** (bottom-center, glassy backdrop-blur UI):

- Move pad (forward/back/left/right) — v0.1
- Rotate pad (yaw + pitch) — v0.1
- Elevate pad (up/down) — **v0.4**
- Zoom pad (+/−) — v0.1
- Held buttons keep applying via an `activeActions` set in the render loop

**Touch / pointer on canvas:**

- 1-finger drag = orbit
- 2-finger pinch = zoom
- 2-finger drag = pan — **v0.3**
- Wheel = zoom

Camera is clamped: distance 2–200, polar angle 0.05 → ~π/2, elevation 1–200.

### UI Chrome

- Top-left info panel with version label and live `cam x, y, z` debug readout — **v0.5**
- Top-right "Generate" button — **v0.6**
- JetBrains Mono font, lime accent color (`#d4ff5f`), safe-area insets for iPad notch/home-bar
- Mobile breakpoint shrinks pad to 36px buttons

### Not Yet Implemented

Compared to the feasibility notes above, still missing: sky/sun shader, shadows, instancing, grass with wind, facade textures.
