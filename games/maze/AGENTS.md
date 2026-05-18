# games/maze — Agent Notes

**File**: `index.html` (~75KB, ~2200 lines, single module script)

## OVERVIEW
3D first-person horror maze. Three.js loaded via importmap from unpkg. One procedural Slenderman that stalks, hunts, teleports behind you, freezes when watched.

## SECTION MAP (in `<script type="module">`)
Find by anchor (line numbers drift). Approximate as of `453399b`:
| Anchor | Approx. line | Section |
|---|---|---|
| `// CONFIG` constants | ~225 | Tunables (maze size, speeds, radii, stamina, catch dist) |
| `function generateMaze` | 304 | Maze gen (iterative recursive-backtracker) |
| `function bfsPath` | ~340 | BFS pathing for monster |
| `function makeBrickTexture` | ~380 | Procedural textures (Canvas2D → CanvasTexture) |
| `class AudioSystem` | 488 | Web Audio (all procedural, no asset files) |
| `const scene = new THREE.Scene()` | 916 | Renderer + camera tree (scene→yaw→pitch→camera) |
| `function buildMaze` | ~960 | Wall geometry + AABB collision array |
| `function makeProceduralMonster` | 1337 | LatheGeometry-based Slenderman |
| `function bfsPath` / `hasLineOfSight` / `isMonsterInPlayerView` | ~1580 | AI spatial helpers |
| `function updateOneMonster` | 1662 | State machine + direct-approach logic |
| `function jumpscareAndDie` | 2085 | Camera snap into killer's face + audio + CSS |
| `(async () => { ... })()` init | 2266 | Bootstrap, GLB load attempt (currently unused), debug=1 handler |

## ARCHITECTURE
- **Camera**: `scene → yaw (Object3D) → pitch (Object3D) → camera`. yaw rotates around Y, pitch around X. NEVER set rotation directly on camera.
- **Monsters**: `monsters[]` array (currently 1 entry). Each `{ root, mixer, role }`. `mixer` holds state-machine fields. Globals `monster`/`monsterMixer` are aliases to `monsters[0]` for legacy back-compat.
- **AI state machine**: `wander | stalk | hunt | stare`. Transitions in `updateOneMonster()`:
  - `forceHunt` (timestamped) overrides everything → `hunt`
  - `staring` (player FOV + LOS) → `stare`, monster speed = 0 then ramps to 22% over 1.2–3.2s (creeping)
  - `heard` (sprint within hear radius) or `veryClose` (≤3 cells) or `directSees` (LOS within 6 cells) → `hunt`
  - else if `distCells ≤ 12` → `stalk`
  - else → `wander`
- **Movement**: cell-grid BFS path (one cell per `MONSTER_REPATH_INTERVAL`). When in same cell as player AND hunt/stare AND `speed > 0.01` → **direct-approach** mode bypasses cell grid, moves world-position directly toward player at 42% of cell speed (capped to leave 5cm clearance).
- **Death**: `checkDeath()` iterates `monsters`. If any `distance < MONSTER_CATCH_DIST (0.95)` → `jumpscareAndDie()`.

## JUMPSCARE PIPELINE
1. Snap `yaw.position` to 38cm in front of killer's head bone (extracted via `monster.userData.head.getWorldPosition()`), camera pitches up at head.
2. Crank flashlight to intensity 18, distance 5m, angle 0.32π.
3. Cut `musicGain` + `ambientGain` to 0 instantly.
4. 55ms tension silence.
5. Fire `audio.scream()` (6-osc dissonant chord + sub-bass + static + reverse swell) + `audio.glassShatter()` (5 detuned squares + 12 high triangles + filtered noise).
6. Add `body.jumpscare-active` class → triggers CSS: canvas saturate/contrast/hue-rotate filter, violent shake animation, red vignette pulsing, blood splatter overlays.
7. Hold 2400ms → remove class → show `YOU DIED` overlay.

## TESTING
- **`?debug=1`** URL param: bypasses menu, places player + monster, enables `forceHuntUntil = +30s`. For visual inspection.
- **`T` key** in-game: cycles through monsters, spawns one 6m in front, sets `forceHuntUntil = +5s`, calls `m.root.lookAt(player)`.
- **`window.__maze`**: exposes `{ monsters(), yaw, CELL, EYE, baseGltf(), gameState(), setGameState(), overlay }` for live inspection from devtools or external scripts.
- **Xvfb visual loop**: software-rendered chromium can't keep up with full GLB loads but handles procedural scenes fine. See root AGENTS.md for the snippet.

## CONVENTIONS
- **Monster geometry = LatheGeometry profiles**, not capsules. Smooth tapered silhouettes only. New body parts: define `[Vector2(radius, y), ...]` profile, `new THREE.LatheGeometry(profile, 18+)`. Skip capsules entirely.
- **All rotations face player via `atan2(monster.x - player.x, monster.z - player.z)`** — sign convention is monster's local -Z faces target, so `atan2(-dx, -dz)` from player-to-monster delta. Easy to get backwards. The motion-direction rotation uses `atan2(-(tx-fx), -(tz-fz))`.
- **Audio routes through submix gains**: `(osc/buffer) → filter → submixGain → master → DynamicsCompressor → destination`. Submix buses: `musicGain` (currently silent), `sfxGain` (events + footsteps + jumpscare), `ambientGain` (currently silent). DO NOT connect directly to `master`.

## ANTI-PATTERNS
- ❌ **No proximity-tracking audio.** `setGrowl()` + `monsterStep()` are kept as no-op stubs. Constant-proximity audio gives away monster location — user explicitly rejected. Random ambient events fire at intervals UNRELATED to monster position.
- ❌ **No drone/pad music.** `_startMusic()` and `_startAmbientBed()` calls are removed from `ensure()`. User feedback: "sine-wave thing — don't want it" and "windy background music sounds bad". The methods still exist in case we want to re-enable for menu only.
- ❌ **No camera shake on proximity.** Removed. Heartbeat + red vignette + threat HUD label convey threat already.
- ❌ **No GLB models for monsters.** Tried CesiumMan (bbox bug), Soldier (looked like armored alien). Procedural LatheGeometry only.
- ❌ **No "stare locks the monster forever" mechanic.** Stare creeps: `mixer.stareTime` accumulates, monster begins moving at up to 22% hunt speed after 1.2s.

## TUNABLES
```
MAZE_W / MAZE_H            16x16 grid (cells of CELL=4 world units)
MONSTER_SPEED_IDLE         1.0 cells/sec (wander)
MONSTER_SPEED_STALK        1.7 cells/sec
MONSTER_SPEED_HUNT         2.9 cells/sec (×CELL = 11.6 m/s peak)
MONSTER_HEAR_RADIUS        9 cells
MONSTER_SIGHT_RADIUS       6 cells
MONSTER_CATCH_DIST         0.95 m
MONSTER_REPATH_INTERVAL    0.4 s
STAMINA_MAX                4.0 s of sprint
```

Direct-approach speed multiplier: `0.42` (in hunt) — keep this < 1.0 or monster reaches player in <0.5s, no reaction window.

## NOTES
- `README.md` in this directory is **outdated** vs current code (claims capsule skeleton, proximity growl, 60 hair strands, 18 cloth strips — all gone). Code is truth.
- `findTeleportCellBehindPlayer()` ray-marches behind the player's facing direction looking for a cell with no LOS from player. Cooldown 11–14s, min distance 5 cells. Currently applies to all monsters (role-gate removed).
- `hasLineOfSight()` is cell-based ray-march at 0.18-unit step size, checks wall presence at each cell boundary crossing.
