# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step — open directly or serve statically:

```bash
open index.html                  # macOS, direct open
python3 -m http.server 8000      # then visit http://localhost:8000
npx serve .
```

## Architecture

Single-page vanilla JS game. All logic lives in `game.js` (~305 lines); `index.html` defines the DOM; `style.css` handles dark/retro styling.

**Core data structures in `game.js`:**
- `board` — `ROWS × COLS` (20×10) 2D array; `0` = empty, `1–7` = piece color index.
- `current` / `next` — piece objects `{ type, shape, x, y }` where `shape` is a 2D matrix.
- `PIECES` — piece shapes defined as square matrices (index 1–7 maps to I, O, T, S, Z, J, L).

**Key functions and their relationships:**
- `collide(shape, ox, oy)` — boundary + overlap check; used everywhere before moving/rotating.
- `tryRotate()` — calls `rotateCW` (transpose + reverse), then tries wall kicks `[0, -1, 1, -2, 2]`.
- `lockPiece()` → `merge()` → `clearLines()` → `spawn()` — the piece-settling chain.
- `spawn()` — promotes `next` to `current`, generates new `next`; triggers `endGame()` if collision on spawn.
- `loop(ts)` — `requestAnimationFrame` loop; accumulates `dropAccum` and drops piece when `≥ dropInterval`.
- `ghostY()` — projects current piece downward until collision; used by both `draw()` and `hardDrop()`.
- `draw()` — clears canvas, draws grid + board + ghost (α=0.2) + current piece each frame.

**Speed formula:** `dropInterval = max(100, 1000 − (level − 1) × 90)` ms. Level increases every 10 lines.

**Scoring:** `LINE_SCORES = [0, 100, 300, 500, 800]` × level. Soft drop +1/row, hard drop +2/cell dropped.

## Tuning parameters

When changing `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `<canvas id="board">` dimensions in `index.html` to match (`COLS × BLOCK` width, `ROWS × BLOCK` height).
