# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No build process, no package manager, no external dependencies — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test step — the project has no `package.json`.

## Architecture

Everything lives in `game.js` as top-level module state and functions (no classes, no modules). Understanding the game requires following the interaction between a few pieces of global mutable state:

- `board`: a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- `current` / `next`: `{ type, shape, x, y }` — `shape` is a square matrix of color indices (see `PIECES`), `x`/`y` are board-cell coordinates of the shape's top-left corner.
- Rotation (`rotateCW`) transposes + reverses rows of `shape` — there is no separate rotation-state table, so wall kicks in `tryRotate` are a simple fixed offset list (`[0, -1, 1, -2, 2]`), not full SRS kick tables.
- Collision (`collide(shape, ox, oy)`) is the single source of truth for "can a shape be at this position" — used for movement, rotation, ghost-piece projection, and spawn/game-over detection alike.

Game loop (`loop`, driven by `requestAnimationFrame`) accumulates elapsed time in `dropAccum` and advances the piece one row whenever it exceeds `dropInterval`; `dropInterval` shrinks as `level` increases (`clearLines`).

Piece lifecycle: `spawn()` promotes `next` to `current` and generates a new `next`; if the newly spawned piece immediately collides, `endGame()` fires. `lockPiece()` (called from soft/hard drop or the natural fall) merges the current piece into `board`, clears completed lines, and calls `spawn()` again.

Rendering is two independent canvases redrawn from scratch every frame: `draw()` for the main board (grid → locked blocks → ghost piece → current piece, in that order) and `drawNext()` for the next-piece preview. There is no dirty-rect tracking.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`. If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` attributes in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
