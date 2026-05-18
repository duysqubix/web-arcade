# Web Arcade

A collection of single-file, zero-install browser games. Each game lives in one HTML file — no bundlers, no build steps, no dependencies to install. Open the file in any modern browser and play.

**Live**: [duysqubix.github.io/web-arcade](https://duysqubix.github.io/web-arcade/)

## Philosophy

Inspired by the [js13kGames](https://js13kgames.com/) tradition. The constraints are deliberate:

- **One file per game.** All HTML, CSS, JavaScript, textures, audio, models — inline.
- **No build step.** What you see in the source is what runs in the browser.
- **No installed dependencies.** Libraries (when used) are loaded from CDN via ES modules or `<script>` tags.
- **Plays from `file://`.** Double-click the HTML file and it works.

## Games

### [The Maze](games/maze/)

A procedurally-generated 3D dungeon maze. First-person horror — something else is in there with you. It hears your footsteps, hunts you through dark corridors using BFS pathfinding, and you only escape by reaching the green beacon at the far corner.

- WASD + mouse · Shift to sprint (but it's loud)
- Procedural skin texture, stringy hair, tattered robe, claw fingers, digitigrade legs
- Procedural audio: heartbeat that quickens with proximity, sub-bass growl, jumpscare screech
- Built with Three.js · ~58 KB total

## Run locally

Any static file server works. Examples:

```bash
python3 -m http.server 8000
```
```bash
npx serve
```
```bash
caddy file-server --listen :8000
```

Then open `http://localhost:8000/`.

Or just **double-click `index.html`** — the games run from `file://` URLs too.

## Contributing a game

1. Fork the repo
2. Create `games/your-game/index.html` (single file, self-contained)
3. Add a card entry to the root `index.html` landing page
4. Update this README
5. Open a PR

Constraints to follow:

- Single HTML file per game
- Inline everything (or CDN-load libraries — no `node_modules`)
- Works offline once the page loads (or document the CDN dependencies)
- Works in current Chrome, Firefox, Safari, Edge

## License

MIT — see [LICENSE](LICENSE)
