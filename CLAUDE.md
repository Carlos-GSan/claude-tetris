# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or serve locally:

```bash
open index.html
# or
python3 -m http.server 8000
```

## Architecture

Three files, no dependencies, no bundler:

- `index.html` — DOM structure: `<canvas id="board">` (300×600px), sidebar panel (score/lines/level/next-piece preview), and `#overlay` for pause/game-over states.
- `style.css` — Dark retro-arcade theme; flexbox layout.
- `game.js` — All game logic (~305 lines, `'use strict'`, ES6+).

### game.js internals

**State**: module-level `let` vars — `board` (ROWS×COLS matrix of color indices 0–7), `current`/`next` piece objects `{type, shape, x, y}`, plus `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`.

**Piece representation**: `PIECES[type]` are 2D matrices where each non-zero cell stores the piece's color index (same value used to index `COLORS[]`).

**Core loop**: `requestAnimationFrame(loop)` → accumulates `dropAccum`; when it exceeds `dropInterval`, drops piece one row or calls `lockPiece()` → `merge()` + `clearLines()` + `spawn()`.

**Collision**: `collide(shape, ox, oy)` checks bounds and board occupancy. Used everywhere: movement, rotation, auto-drop, spawn (triggers game over if true).

**Rotation**: `rotateCW` (transpose + reverse rows); `tryRotate` applies wall kicks `[0, -1, 1, -2, 2]`.

**Canvas sizing**: `canvas` is fixed at `COLS × BLOCK` × `ROWS × BLOCK` (300×600). If `COLS`, `ROWS`, or `BLOCK` constants change, update `width`/`height` on `<canvas id="board">` in `index.html` to match.

**Speed formula**: `dropInterval = max(100, 1000 − (level − 1) × 90)` ms; level increments every 10 lines.
