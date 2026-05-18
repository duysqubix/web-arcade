# web-arcade — Agent Notes

**Commit**: `453399b` · **Branch**: `main`

## OVERVIEW
Collection of single-file HTML browser games. Each game is one self-contained HTML file with all CSS/JS/assets inline; libraries from CDN only (no `node_modules`). Plays from `file://` or any static server.

## STRUCTURE
```
.
├── index.html          # Landing page (cards linking to games/)
├── games/
│   └── maze/           # See games/maze/AGENTS.md
└── README.md           # Player-facing intro
```

`games/` is a passthrough directory. New games go in `games/<name>/index.html`.

## WHERE TO LOOK
| Task | Location |
|---|---|
| Add a new game | `games/<name>/index.html` + card entry in root `index.html` |
| Adjust landing visuals | Root `index.html` (CSS at top, SVG thumbnails inline per card) |
| Adjust game mechanics | `games/<game>/index.html` only — never split across files |
| Game-specific agent notes | `games/<game>/AGENTS.md` (each game has its own) |

## CONVENTIONS
- **One HTML file per game.** No `.css`/`.js` split. No build step. Inline `<style>` and `<script type="module">`.
- **External libs only via CDN.** Use `<script type="importmap">` for ES module mapping (see `games/maze/index.html` head).
- **Procedural assets only.** Textures via Canvas2D → `THREE.CanvasTexture`. Audio via Web Audio API oscillators/buffers. No external image/audio files.
- **Tailscale-hosted local dev**: server runs on `pop-os` over Tailscale at `:8765`. Real public URL = `pop-os.moray-gila.ts.net:8765`.
- **Commit footer**: standard `git commit`, no co-author lines required.

## ANTI-PATTERNS
- ❌ `package.json` / `node_modules` / npm install steps. Repo must stay zero-install.
- ❌ External `.glb`/`.png`/`.mp3` asset files. Embed or generate.
- ❌ Build steps (webpack, vite, esbuild). What's in source must run unchanged.
- ❌ Loading CORS-restricted CDNs. Use `unpkg`, `jsdelivr`, `raw.githubusercontent` — all send `access-control-allow-origin: *`.

## COMMANDS
```bash
tmux new-session -d -s web-arcade -c ~/.repos/web-arcade \
  'python3 -m http.server 8765 --bind 0.0.0.0'

tmux kill-session -t web-arcade

python3 -c "import re; c=open('games/maze/index.html').read(); m=re.search(r'<script type=\"module\">(.*?)</script>',c,re.DOTALL); open('/tmp/m.mjs','w').write(m.group(1))" && node --check /tmp/m.mjs

pkill -9 Xvfb; Xvfb :99 -screen 0 1280x720x24 -ac >/dev/null 2>&1 &
DISPLAY=:99 ~/.cache/ms-playwright/chromium-1223/chrome-linux64/chrome \
  --no-sandbox --use-angle=swiftshader --enable-unsafe-swiftshader \
  --user-data-dir=/tmp/cd-test --no-first-run --kiosk \
  --window-size=1280,720 "http://localhost:8765/games/maze/?debug=1" &
sleep 5; DISPLAY=:99 import -window root /tmp/snap.png
```

## NOTES
- **Repo lives at** `~/.repos/web-arcade` (moved from `~/web-arcade`).
- **GitHub remote** uses the `githubqubix` SSH alias (`~/.ssh/config` Host entry → `~/.ssh/id_ed25519`).
- **Token caveat**: `gh auth token` is a fine-grained PAT without `Administration: write` — cannot create repos / change default branch / enable Pages via API. User must do those via web UI.
- **GitHub Pages**: not yet enabled. Setting up requires `github.com/<user>/<repo>/settings/pages` — manual click. Once on, served at `https://duysqubix.github.io/web-arcade/`.
- **Game READMEs may be stale.** Code is source of truth, README is player-facing copy that drifts. Verify implementation details in the HTML source, not the README.
