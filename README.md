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

## Current State (v0.26)

### Rendering / Scene

- Three.js (r166) loaded from CDN, single-file ES module
- WebGL renderer, sRGB output, `devicePixelRatio` capped at 1.5
- Brighter `#de5b68` sky with a darker `#873a42` fog — near/far computed each `generateCity()` from real 3D camera-to-corner distances; `fog.far` extends past the city so the farthest buildings stay as hazed silhouettes instead of fully erasing — **v0.26**
- Camera `far` plane bumped to 3000 so the ground disc renders all the way to its real edge and fades into fog instead of being clipped mid-air — **v0.26**
- Ambient + directional lighting (no shadows yet)
- `#54163d` circular ground disc (1600-unit diameter cylinder, 1 unit thick, top flush with `y=0`) — **v0.21**
- Camera starts further back at `(250, 175, 250)`; `MAX_DISTANCE` bumped to 700 — **v0.22**

### Procedural City

- Grid oversized to ~2.2× target buildings so some lots stay empty and multi-cell footprints fit — **v0.8**
- Per-row / per-column road widths jittered around the base 3-unit spacing (`ROAD ± 1.6`) — **v0.8**
- Variable footprints: ~55% 1×1, ~15% 2×1, ~15% 1×2, ~15% 2×2, so some buildings are squares, some rectangles, some big blocks — **v0.8**
- Random cell-fill fraction per building (small lots 55–90%, larger lots 78–96%)
- Random heights bucketed into four tiers — skyscrapers (50–80, 2× tall), tall (25–40), medium (8–15), small (3–7) — with distribution controlled by sliders — **v0.22**
- All buildings share a single `MeshStandardMaterial` tinted `#ed2651` — **v0.9**
- "Generate" button reseeds the layout

### Generation Settings (v0.7)

- Collapsible panel (chevron toggle), defaults collapsed so only Generate + chevron are visible
- **Buildings** slider: total count 1–1000 (default 1000) — **v0.18**
- **Skyscrapers** / **Tall** / **Medium** sliders: percentages, scaled-down proportionally when their sum would exceed 100 (defaults 5% / 17% / 45%, leaving 33% small) — **v0.21**
- **Small** slider: disabled, auto-updates to `100 − skyscrapers − tall − medium`
- Expanded panel `max-height` is 1500px and has 10px `padding-bottom` so the last slider's thumb isn't clipped by the `overflow: hidden` used for the collapse animation — **v0.23**
- Scene only regenerates when Generate is pressed; sliders just update labels

### Camera Controls

Two parallel input paths feeding the same camera math (`orbit`, `zoom`, `pan`, `elevate`):

**On-screen pad** (bottom-center, glassy backdrop-blur UI, collapsible via a chevron toggle and starts collapsed — **v0.24/v0.26**):

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
