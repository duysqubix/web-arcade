# The Maze

A procedurally-generated 3D dungeon maze with a stalking monster. Single HTML file, ~58 KB.

**Play**: open `index.html` in any modern browser, or visit [duysqubix.github.io/web-arcade/games/maze/](https://duysqubix.github.io/web-arcade/games/maze/)

## Controls

| Key | Action |
|---|---|
| W A S D / Arrows | Move |
| Mouse | Look |
| Shift | Sprint (loud — wakes the monster) |
| N | Generate a fresh maze (mid-game) |
| Esc | Pause / release mouse |

## How it works

| System | Implementation |
|---|---|
| Maze generation | Iterative recursive-backtracker, 16×16 grid, perfect maze (every cell reachable) |
| Rendering | Three.js loaded from CDN, ES module import, WebGL renderer |
| Textures | All procedural — drawn at boot onto Canvas2D contexts and wrapped as `CanvasTexture` |
| Wall collision | AABB checks against an array of wall rectangles with player radius padding |
| Monster AI | State machine: `wander` (random walk) → `hunt` (BFS pathfinding to player cell), re-pathed every 0.45s |
| Monster aggro | Sees player within 6 cells, hears player sprinting within 8 cells |
| Monster anatomy | Capsule-based skeleton with parented limb groups for walk animation. Skull is a sphere with manually-displaced vertices (sunken cheeks, brow ridge, pointed chin). 60 individual hair strand cylinders. 18 cloth strip planes. |
| Walk animation | Out-of-phase hip + shoulder rotations driven by `performance.now()` phase, amplitude scaling with `hunt` state |
| Audio | Web Audio API, fully procedural. Ambient brown-noise rumble through low-pass filter. Sub-bass growl with LFO frequency modulation, volume ramped by proximity. Heartbeat = sine kick. Footsteps = filtered noise burst. Death screech = stacked saw oscillators sweeping down. |
| Stamina | 4-second budget, drains while sprinting, refills slowly. Empty → exhausted lockout until 40% recovery. |
| Threat indicator | Distance-bucketed labels (`SILENT` → `WHISPERS` → `BREATHING` → `CLOSE` → `BEHIND YOU` → `RUN.`) and radial red vignette |
| Death | Distance < 0.85 → jumpscare overlay (procedural Canvas2D face: sunken sockets, glowing pupils, jagged teeth, blood streaks) with CSS shake animation + 1.6s multi-octave screech |
| Effects | Persistent SVG film grain overlay with `mix-blend-mode: overlay` + CRT scanlines |

## Configuration

All gameplay constants live near the top of the `<script type="module">` block:

```js
const MAZE_W = 16;             // maze width in cells
const MAZE_H = 16;             // maze height in cells
const MONSTER_SPEED_IDLE = 1.4;
const MONSTER_SPEED_HUNT = 2.6;
const MONSTER_HEAR_RADIUS = 8; // cells
const MONSTER_SIGHT_RADIUS = 6;
const MONSTER_CATCH_DIST = 0.85;
const STAMINA_MAX = 4.0;       // seconds of sprint
```

Crank `MAZE_W` and `MAZE_H` for a bigger labyrinth. The recursive backtracker is iterative so it won't blow the stack.

## Dependencies

- [Three.js r160](https://unpkg.com/three@0.160.0/build/three.module.js) — loaded from unpkg CDN

That's it. No npm, no bundler, no build.
