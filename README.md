# Gummy/Bears.

**English** · [繁體中文](README.zh-TW.md)

![A table of translucent gummy bears](docs/screenshots/hero.png)

## Introduction

*Gummy/Bears.* is a small interactive toy: a studio table full of translucent, wobbly gummy bears that you can grab by the ear, paw, snout or head. Stretch them into taffy, squash them into pancakes and launch them, or flick them across the table so they bounce off each other.

There isn't a single block in it. There are no polygon meshes, voxels, sprites or texture images. The GPU draws one full-screen triangle, and every pixel is **raymarched from signed distance functions (SDFs)**. Each bear is a smooth blend of rounded cones that are moved every frame by a custom spring-mass simulation.

- One self-contained HTML file: HTML, CSS, GLSL and JavaScript together
- No frameworks, no build step, and no external resources such as fonts, CDNs or images
- Runs offline in any browser with WebGL2

## Quick start

**Option 1: open the file directly**

Double-click `index.html` (it forwards to `gummy-bears.html`) or open `gummy-bears.html` itself in Chrome, Edge, Firefox or Safari.

**Option 2: use a static server**

```bash
# any of these, from the project folder
python3 -m http.server 8000
npx serve .
```

Then open <http://localhost:8000/>.

> Requires WebGL2. If it isn't available, the page shows a short notice instead of the scene.

## Screenshots

| Pull an ear | Pull the head up | Press the head down |
| :---: | :---: | :---: |
| ![Ear stretched into taffy](docs/screenshots/stretch-ear.png) | ![Neck stretch](docs/screenshots/neck-stretch.png) | ![Pancake](docs/screenshots/pancake.png) |

| Pour more bears | Mobile |
| :---: | :---: |
| ![Bears pouring onto the table](docs/screenshots/pour.png) | <img src="docs/screenshots/mobile.png" alt="Mobile layout" width="260"> |

## Controls

| Input | Action |
| --- | --- |
| **Drag** an ear, paw, foot or snout | Stretch that part (it stays attached to the body) |
| **Drag the head up** | Stretch the neck; release to hop |
| **Drag the head down** | Squash the bear into a pancake; release to launch it |
| **Drag the belly / chest** | Pick the whole bear up and carry it |
| **Release while moving** | Flick: the bear is thrown with the pointer's momentum |
| Click a **flavor** swatch | Recolor the last bear you touched (or every bear, if you haven't touched one yet); new pours use this flavor |
| **Wobble** slider | 0% is firm and quickly damped; 100% is very jiggly |
| **Pour more bears** | Drop 5 new bears from above (up to 20; the oldest fade out) |

Mouse, touch and pen all work. On desktop the pointer is a thin ring with a centre dot that shrinks when you grab, squashes while you press, and stretches in the direction you pull.

## Gameplay

The bears follow four interactions, listed at the bottom left of the page:

1. **Pull limb / ear: stretch & wobble.** The pulled part stretches into a gelatin bridge that thins as it lengthens. The pull travels through the bear's springs, so it leans toward you. Let go and the part snaps back like a rubber band, and the recoil sets the whole bear wobbling.
2. **Pull head up: neck stretch & hop.** The chest and belly are drawn up with the head and the neck elongates. On release the bear springs into a hop.
3. **Press head down: pancake & launch.** All nodes spread outward and flatten into a wide pancake (volume is preserved), storing elastic energy. On release the bear launches high into the air, spinning.
4. **Flick: throw & scatter.** Release any part mid-motion to throw the bear. It bounces several times with a squash on each landing, rebounds off the walls, and bumps softly into other bears.

The card on the right shows two live numbers:

- **STRETCH:** the current largest stretch ratio on the table.
- **BOUNCES:** how many landings have happened so far.

A bear that topples over slowly rights itself back onto its feet.

## Rendering

**Geometry (SDF)**

- Each bear has 10 hierarchical nodes: belly (root), chest, head, snout, left/right ears, left/right arms and left/right legs.
- Every non-root node is an exact **round-cone SDF** drawn from its parent to itself, and all cones are merged with a polynomial **smooth-min**. Ear↔head, snout↔head, head↔chest, arms↔chest and legs↔belly are always welded together, so no part can break off into a separate sphere.
- **Volume-preserving stretch:** every bone's radii scale by `1 / sqrt(length / restLength)`.
- **Pancake squash:** a non-uniform scale `(1/√s, s, 1/√s)` is applied in SDF space (it preserves volume), together with the physical spreading of the nodes.
- The glossy dark eyes and nose are tiny separate spheres with their own material.

**Gelatin material**

- **Refraction** (index of refraction 1.45) into the body. A second march inside the SDF measures thickness, the ray refracts again on exit, and whatever lies behind (the floor and its shadows) is shaded.
- **Beer–Lambert absorption:** `exp(-σ·thickness)`, where `σ` comes from the flavor color. Thin ears glow and thick bellies go deep and saturated.
- **Subsurface scattering:** single scattering at the middle of the path, attenuated by the distance the light travels through the gelatin from the key softbox.
- **Micro-bubbles:** a hashed 3D cell field sampled at several depths in the bear's local frame, so the bubbles move and rotate with the bear.
- **Fresnel** (Schlick) reflections of a studio environment, with **two rectangular softboxes** for crisp highlights.

**Studio floor**

- An off-white `#f5f5f3` surface that blends seamlessly into the page.
- **Colored soft shadows** from analytic sphere soft-shadows on every node and joint, tinted by light passing through the gelatin.
- **Caustics:** bright focused lobes under the torso and head.
- **Contact ambient occlusion:** analytic sphere occlusion with a hint of the bear's color.

**Performance**

- Per-pixel bounding-sphere culling: each pixel only marches the bears its ray can hit.
- Bear data lives in a uniform buffer.
- The highlight shoulder is applied only above the background brightness, so the page color stays exact.
- **Adaptive render resolution** responds to frame time.

**Physics**

- Fixed 240 Hz steps.
- **Shape matching** with full rotation, using quaternion polar extraction (Müller et al.).
- A **two-way parent↔child bone spring network** that stiffens like taffy when stretched.
- Contact-anchored support so a standing bear carries its own weight.
- Whole-body rubber bounces with landing squash.
- Soft node-vs-node bear collisions.
- Trapezoidal walls that match the visible table.

## Project structure

```
my-bear-jelly/
├── index.html            # entry point: forwards to gummy-bears.html
├── gummy-bears.html      # the whole app, in one file
│   ├── <style>           #   editorial UI, control card, custom cursor, responsive layout
│   ├── <script id="vs">  #   full-screen triangle vertex shader
│   ├── <script id="fs">  #   SDF raymarcher: bears, gelatin material, floor, caustics
│   └── <script>          #   physics, picking and interaction, UI wiring, render loop
├── docs/
│   └── screenshots/      # images used in this README
├── README.md             # English
└── README.zh-TW.md       # 繁體中文
```
