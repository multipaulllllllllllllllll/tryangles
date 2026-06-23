# TRIANGULATE

> Claim edges. Complete triangles. Beat the bot.

A real-time competitive edge-claiming game built with Three.js. Open `triangle-game.html` directly in your browser — no build step, no dependencies to install.

**Live:** `https://multipaulllllllllllllllll.github.io/tryangles`

---

## How to Play

1. **Click any edge** (line between two vertices) to claim it — it turns cyan
2. **Complete a triangle** (claim all 3 edges) = **+1 point + bonus turn**
3. **The bot** (cardinal red) takes turns after you
4. **Fill 60%** of available triangles before time runs out → advance to next stage
5. **Time expires** → whoever has more completed triangles wins

---

## Stages

| Stage | Triangles | Time |
|-------|-----------|------|
| 1 — Warm Up | 4 | 60s |
| 2 — Getting Busy | 9 | 70s |
| 3 — Serious | 16 | 80s |
| 4 — Master | 25 | 90s |

---

## Bot AI

- **Stages 1–2** — random edge selection
- **Stages 3–4** — greedy: completes its own triangles first, then blocks your open triangles

---

## Controls

| Input | Action |
|-------|--------|
| Click edge | Claim edge (your turn) |
| Scroll | Zoom in/out |
| Drag | Rotate view |

---

## Project Structure

```
tryangles/
├── SPEC.md            — Full technical specification
├── triangle-game.html  — The game (single self-contained file)
└── README.md          — This file
```

---

## Tech Stack

- **Three.js 0.163.0** — 3D rendering (CDN)
- **OrbitControls** — camera zoom/rotate
- **Vanilla HTML/CSS/JS** — no build tools
- **JetBrains Mono + Archivo Black** — typography via Google Fonts

---

## Dev Notes

Open `triangle-game.html` in a browser. Use **F12 → Console** for debug output. The file is ~1,380 lines — intentionally kept as a single artifact for easy sharing and zero-friction deployment.
