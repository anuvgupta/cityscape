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

## Current State (v0.7)

### Rendering / Scene

- Three.js (r166) loaded from CDN, single-file ES module
- WebGL renderer, sRGB output, `devicePixelRatio` capped at 1.5
- Dark scene (`#0a0a14`) with fog (80 → 250 units)
- Ambient + directional lighting (no shadows yet)
- Grid helper + axes helper for reference

### Procedural City

- Grid sized from settings: `COLS = ceil(√total)`, `ROWS = ceil(total/COLS)`, 8-unit cells with 3-unit roads — **v0.7**
- Random heights bucketed into three tiers — skyscrapers (25–40), medium (8–15), small (3–7) — with distribution controlled by sliders — **v0.7**
- Random footprint per building (55–90% of cell)
- All buildings share a single `MeshStandardMaterial`
- "Generate" button reseeds the layout

### Generation Settings (v0.7)

- Collapsible panel (chevron toggle), defaults collapsed so only Generate + chevron are visible
- **Buildings** slider: total count 1–500 (default 20)
- **Skyscrapers** / **Medium** sliders: percentages, mutually clamped so their sum ≤ 100
- **Small** slider: disabled, auto-updates to `100 − skyscrapers − medium`
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
