# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step or dependencies. Open directly or serve statically:

```bash
open index.html                  # macOS — open directly
python3 -m http.server 8000      # local server, then visit http://localhost:8000
```

## Architecture

Three files, no framework:

- **`index.html`** — DOM structure: a `300×600` `<canvas id="board">` for the playfield, a `120×120` `<canvas id="next-canvas">` for the next-piece preview, a sidebar HUD (`#score`, `#lines`, `#level`), and an `#overlay` div for PAUSE/GAME OVER states.
- **`style.css`** — Dark/retro arcade theme. Flexbox layout, monospace HUD, `backdrop-filter` overlay.
- **`game.js`** — All game logic (~305 lines, `'use strict'`, no modules).

### Key data structures in `game.js`

- **`board`**: `ROWS × COLS` (20×10) integer matrix. `0` = empty; `1–7` = color index matching `COLORS` and `PIECES`.
- **Piece object**: `{ type, shape, x, y }` where `shape` is a 2D matrix copy from `PIECES[type]`.
- **Global mutable state**: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `lastTime`, `animId`.

### Game loop flow

```
init() → spawn() → requestAnimationFrame(loop)
loop(ts): accumulate dt → auto-drop or lockPiece() → draw() → rAF(loop)
lockPiece(): merge() → clearLines() → spawn()
spawn(): collision on entry → endGame()
```

### Tunable constants

| Constant | Default | Effect |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Board size — must also update canvas `width`/`height` in HTML (`COLS×BLOCK` × `ROWS×BLOCK`) |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Color per piece type (index 1–7) |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points for 1–4 cleared lines (multiplied by level) |
| `dropInterval` | 1000 ms | Initial fall speed; recalculated as `max(100, 1000 − (level−1)×90)` |
