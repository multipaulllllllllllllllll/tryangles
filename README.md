# Tryangle

> Claim edges on 3D shapes. Race the bot. Complete the faces.

A real-time edge-claiming game built with Three.js. Open `tryangle.html` in your browser — no build step, no dependencies to install.

**Live:** [multipaulllllllllllllllll.github.io/tryangles](https://multipaulllllllllllllllll.github.io/tryangles)

---

## How to Play

1. **Click any edge** to claim it (cyan = you, red = bot)
2. **Own all 3 edges** of a triangular face to score +1
3. **Race in real time** — the bot claims edges on a timer (no turns)
4. **Fill 60%** of faces to advance, or have the lead when time runs out
5. **Clear all edges** with the lead to advance early

---

## Stages

| Stage | Shape | Faces | Time | Bot interval |
|-------|-------|-------|------|--------------|
| 1 | Tetrahedron | 4 | 60s | 3.5s |
| 2 | Octahedron | 8 | 70s | 2.5s |
| 3 | Icosahedron | 20 | 80s | 1.8s |
| 4 | Subdivided icosahedron | 80 | 90s | 1.0s |

---

## Controls

| Input | Action |
|-------|--------|
| Click edge | Claim edge |
| Scroll | Zoom |
| Drag | Rotate view |
| ♪ ON/OFF | Toggle 8-bit music & SFX |

---

## Project Structure

```
tryangles/
├── tryangle.html       — Current game (single self-contained file)
├── triangle-game.html  — Earlier turn-based prototype
├── index.html          — GitHub Pages entry → tryangle.html
├── SPEC.md             — Original technical spec (partially outdated)
└── README.md           — This file
```

---

## Tech Stack

- **Three.js 0.163.0** — 3D polyhedra, orbit camera (CDN)
- **Web Audio API** — procedural 8-bit music and sound effects
- **Vanilla HTML/CSS/JS** — no build tools
- **Press Start 2P + Orbitron** — arcade typography via Google Fonts

---

## Run Locally

```bash
python3 -m http.server 8000
# → http://localhost:8000/tryangle.html
```

Or open `tryangle.html` directly in Chrome, Firefox, or Safari.