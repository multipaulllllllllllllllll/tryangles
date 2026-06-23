# TRIANGULATE — Technical Specification

## Overview

A real-time competitive edge-claiming game built as a single self-contained HTML file using Three.js. Two players (human + bot) compete to complete triangles on a triangular lattice by clicking/claiming edges. The player with more completed triangles when time expires wins.

**File:** `triangle-game.html` (~1,380 lines, ~43KB)
**Stack:** Vanilla HTML/CSS/JS + Three.js 0.163.0 (CDN)
**Target:** Desktop browsers (Chrome, Firefox, Safari)

---

## Visual Style

| Element | Value |
|---|---|
| Background | `#09090b` |
| User color | `#00d4ff` (cyan) |
| Bot color | `#C8102E` (cardinal red) |
| Free edge | `#3a3a44` (dim grey) |
| Hover edge | `#6a6a7a` (lighter grey) |
| Font (UI/titles) | Archivo Black |
| Font (mono) | JetBrains Mono |

**Scene:** Perspective camera, `FogExp2` depth atmosphere, directional + ambient lighting. OrbitControls with damping for scroll-to-zoom and pan-to-rotate.

---

## Architecture

### File Structure

```
triangle-game.html
├── <head>           — CSS variables, all styles, Google Fonts
├── <body>           — HUD, overlays, screen divs, canvas-wrap
├── <script type="importmap">  — Three.js CDN imports
└── <script type="module">    — All game code (Three.js module)
```

### Three.js Scene Hierarchy

```
scene
├── latticeGroup      — vertex dots, edge LineSegments, triangle fills
├── edgeLineGroup     — fat wireframe edges (LineSegments)
├── triFillGroup      — filled triangle meshes (ShapeGeometry)
└── [fog atmosphere]
```

### State Object

```javascript
state = {
  stage: 0,           // 0-3
  userScore: 0,
  botScore: 0,
  turn: 'user',       // 'user' | 'bot'
  isGameOver: false,
  timer: 60,
  timerInterval: null,
  bonusTurn: false,
  edges: [],          // { id, vi, vj, from, to, owner }
  triangles: [],      // { id, vi, vj, vk, _owner }
  triangleMeshes: [],  // { mesh, triId, owner }
  hoveredEdge: null,
  // populated by buildLatticeForStage:
  verts: [],
  edgeMap: Map,       // "minV-maxV" → edgeId (from buildLattice)
  edgeByVerts: Map,   // "vi-vj" ↔ "vj-vi" → edgeId (rebuilt in buildLatticeForStage)
}
```

---

## Triangular Lattice Geometry

### Vertex Layout (n = lattice order)

For a lattice of order `n` (n² total triangles):

- Row `r = 0..n` (n+1 rows total)
- Row `r` even: `n+1` vertices at `x ∈ [-n/2, n/2]`, y-pos = `-r × √3/2`
- Row `r` odd: `n` vertices at `x ∈ [-n/2 + 0.5, n/2 - 0.5]`, same y-pos

### Edge Types

1. **Horizontal** — each row connects adjacent vertex pairs
2. **Slanted down-left** — even row vertex `i` → odd row vertex `i-1` (if valid)
3. **Slanted down-right** — even row vertex `i` → odd row vertex `i` (if valid)
4. **Slanted up-right** — odd row vertex `j` → even row vertex `j` (always)
5. **Slanted up-left** — odd row vertex `j` → even row vertex `j+1` (always)

Deduplication via `edgeMap` (`"minV-maxV"` key). Each edge stored once with `from`/`to` world coordinates.

### Triangle Types (n² total)

- **Type-A (downward-pointing):** `n` triangles per even row `r` where `0 ≤ r < n`. Base on even row `r`, apex on odd row `r+1`.
- **Type-B (upward-pointing):** `n` triangles per odd row `r` where `1 ≤ r < n`. Apex on odd row `r`, base on even row `r+1`.

Vertex indices stored as `vi, vj, vk` on each triangle object.

### Stage Configuration

| Stage | Lattice n | Triangles | Time | Threshold |
|---|---|---|---|---|
| 1 Warm Up | 2 | 4 | 60s | 60% |
| 2 Getting Busy | 3 | 9 | 70s | 60% |
| 3 Serious | 4 | 16 | 80s | 60% |
| 4 Master | 5 | 25 | 90s | 60% |

---

## Rendering

### Edge Rendering (Fat Wireframe)

Each edge is rendered via:
1. **LineSegments** — `BufferGeometry` with `position` + `color` attributes. Updated in-place when ownership changes via `updateEdgeColor()`. `LineBasicMaterial` with `vertexColors: true`.
2. **Invisible Picking Sphere** — `SphereGeometry` centered at edge midpoint, radius = `len/2 + 0.8 × EDGE_PICK_SIZE`. `MeshBasicMaterial({ visible: false })`. `OrbitControls` passes through without interfering.

Why spheres over cylinders: diagonal edges in a triangular lattice have arbitrary angles. A cylinder oriented via `CylinderGeometry.rotateZ()` doesn't reliably align with all edge directions. A sphere at the midpoint is approach-angle invariant and guarantees correct hit detection.

**Edge pick size:** `EDGE_PICK_SIZE = 0.18`

### Vertex Dots

`CircleGeometry(0.07, 12)` at each vertex position, `z = 0.02`. Dim grey `#4a4a56`, opacity 0.8.

### Triangle Fill

`ShapeGeometry` built from 3 vertex offsets from the triangle centroid. `MeshBasicMaterial`, `transparent: true`, `depthWrite: false`, `side: THREE.DoubleSide`.
- Inactive: `opacity = 0`
- Active (owned): `opacity = 0.40`, color = owner (cyan or red)

Updated on `claimEdge()` via `updateTriangleFill()`.

---

## Game Logic

### Claiming an Edge

```javascript
claimEdge(edgeId, owner) → tid[]
```

1. Set `state.edges[edgeId].owner = owner`
2. Call `updateEdgeColor(edgeId, owner)` — updates LineSegments color buffer in-place
3. Call `checkNewTriangles(owner)` — scans all triangles for newly completed ones, sets `t._owner`, returns array of completed triangle IDs
4. For each new triangle: `updateTriangleFill(tid, owner)` + `triggerFlash(owner)`
5. Returns array of newly completed triangle IDs

### Turn Flow

**User clicks edge:**
1. `claimEdge(edgeId, 'user')` → `newTris`
2. `endUserTurn(newTris)`:
   - If `newTris.length > 0`: bonus turn, update score, set turn stays `'user'`
   - If `newTris.length === 0`: turn → `'bot'`, `setTimeout(botTurn, 900-1500ms)`
3. `updateScoreDisplay()` + `checkStageAdvance()`

**Bot turn:**
1. `chooseBotEdge()` → `edgeId`
2. `claimEdge(edgeId, 'bot')` → `newTris`
3. `endBotTurn(newTris)`:
   - If `newTris.length > 0`: bonus turn, score, `setTimeout(botTurn, 700-1500ms)` (guarded by `!state.isGameOver`)
   - If `newTris.length === 0`: turn → `'user'`

### Bot AI

| Stages | Strategy |
|---|---|
| 1–2 (warm) | Random from available edges |
| 3–4 (serious+) | 1. Complete own triangles (2 bot-owned edges + 1 free) |
| | 2. Block user triangles (2 user-owned edges + 1 free) |
| | 3. Prefer edges touching the most triangles |

### Win/Loss Conditions

- **Advance:** User completes `≥ threshold × total` triangles → `showScreen('advance-screen')`
- **Timeout:** Timer hits 0 → `onTimeUp()`:
  - User triangles > Bot triangles → `showScreen('win-screen')`
  - Bot triangles > User triangles → `showScreen('timeout-screen')`
  - Tie → `showScreen('timeout-screen')`
- **Lose screen:** Shown when user loses at stage end (time or bot wins count)

---

## UI / HUD

### Screens

| Screen | Trigger | Buttons |
|---|---|---|
| `start-screen` | Initial load | `startGame()` |
| `advance-screen` | User hits threshold | `nextStage()` |
| `win-screen` | Win at any stage / timeout win | `restartGame()` |
| `lose-screen` | (unused — timeout uses timeout-screen) | `restartGame()` |
| `timeout-screen` | Time expires, user didn't win | `restartGame()` |

All screens use `.screen.hidden { opacity: 0; pointer-events: none; }` to hide/show.

### HUD Elements

- `#hud-title` — game name top-left
- `#stage-label` — current stage top-center
- `#timer-display` — seconds remaining (red when ≤10, amber when ≤20)
- `#score-panel` — `You: N : M Bot`
- `#bottom-bar` — turn indicator + progress bar fill + percentage

### Turn Indicator

CSS classes on `#turn-indicator`:
- `.player` — cyan text "Your Turn"
- `.bot` — red text "Bot's Turn"
- `.bonus` — amber text "+N Bonus!"

---

## Controls

- **Click edge** — claim it (user turn only)
- **Scroll** — zoom in/out (`OrbitControls` with damping, min=2, max=60)
- **Drag** — rotate view (`OrbitControls`)
- **Touch** — `touchstart` listener mirrors click logic

---

## Known Issues / Open Items

1. **Center edge clicks may reset game** — reported by user; investigation ongoing. Picking uses sphere geometry (fixed from cylinder). Debug logging added to click handler. Root cause not yet confirmed — possibly related to `edgeMeshes` array state during stage transitions.
2. **LineBasicMaterial linewidth** — WebGL spec guarantees minimum visible width of 1 but actual rendered thickness is driver-dependent. On some systems edges may appear thinner than expected.
3. **No sound** — audio not implemented.
4. **Mobile** — touch support exists but UI is designed for desktop viewport.
5. **Git history** — initial commit at `3f2656b`, second commit at `143e801`.

---

## Build & Run

```bash
# Open directly in browser (no build step)
open triangle-game.html

# Or serve locally
python3 -m http.server 8000
# → http://localhost:8000/triangle-game.html
```

**No dependencies to install.** Three.js and fonts load from CDN on first load.

---

## Deployment

Remote: `https://github.com/multipaulllllllllllllllll/tryangles`

```bash
git add triangle-game.html
git commit -m "message"
git push
```
