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
