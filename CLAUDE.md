# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build, no tests, no linter. UI text and README are in Spanish — keep user-facing strings in Spanish.

## Run

```bash
python3 -m http.server 8000   # then open http://localhost:8000
# or just: xdg-open index.html
```

## Architecture

Three files: `index.html` (DOM + two canvases + overlay), `style.css` (dark theme), `game.js` (all logic, classic script — no modules).

Key points in `game.js`:

- **Global mutable state** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`…) declared at top-level; `init()` resets all of it and is also the restart handler. Any new state must be reset in `init()`.
- **Cell values = color index.** `board[row][col]` holds `0` or `1–7`; piece shapes in `PIECES` store their own type number in filled cells, so `merge()` copies shape values straight into the board and `drawBlock` looks up `COLORS[value]`. New piece types need matching entries in both `PIECES` and `COLORS` (and `randomPiece` hardcodes `* 7`).
- **`collide(shape, x, y)`** is the single source of truth for movement, rotation (`tryRotate` with kicks `[0,-1,1,-2,2]`), ghost projection (`ghostY`), drops and game-over detection (collision on `spawn`). Cells with `y < 0` are allowed.
- **Lifecycle:** `lockPiece()` = `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (promotes `next`, may call `endGame()`).
- **Loop:** `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum`; gravity step when `≥ dropInterval`; full redraw every frame. Pause/game-over stop it via `cancelAnimationFrame(animId)`; resume resets `lastTime` and calls `loop` directly.
- **HUD** is DOM (`updateHUD()`), not canvas; call it after any score/lines/level change.

## Coupled constants

- `COLS × BLOCK` / `ROWS × BLOCK` must match `<canvas id="board">` width/height in `index.html` (300×600).
- `drawNext` assumes a 4×4 grid of 30px cells (`NB`), matching `<canvas id="next-canvas">` 120×120.
