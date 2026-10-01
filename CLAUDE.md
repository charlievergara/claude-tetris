# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

There is no build/lint/test tooling. To run:

```bash
open index.html                # macOS, opens directly in browser
python3 -m http.server 8000    # or serve locally, then visit http://localhost:8000
```

Changes to `game.js`, `index.html`, or `style.css` take effect on browser reload — no compilation step.

## Architecture

Everything lives in three files with no module system (plain `<script src="game.js">`):

- **`index.html`** — DOM structure: the `#board` canvas (300×600, i.e. `COLS × BLOCK` by `ROWS × BLOCK`), the `#next-canvas` preview canvas, HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` for pause/game-over.
- **`style.css`** — dark/retro visual theme only; no layout logic depends on it.
- **`game.js`** — all game state and logic, using module-level `let`/`const` globals (no classes, no state container object).

### Core model

- Board is a `ROWS × COLS` matrix (`board[row][col]`), where `0` = empty and `1–7` = a color index into `COLORS`/`PIECES`.
- Pieces (`PIECES`) are defined as fixed square matrices (e.g. I is 4×4, T/S/Z/J/L are 3×3, O is 2×2). Rotation (`rotateCW`) is a generic transpose+reverse — it works on any square shape, so don't special-case individual piece types.
- `current` and `next` are piece objects: `{ type, shape, x, y }`. `spawn()` promotes `next` to `current` and generates a new `next`; if the new `current` immediately collides, `endGame()` fires.

### Key functions to know before modifying behavior

- `collide(shape, ox, oy)` — the single source of truth for out-of-bounds/overlap checks. Any new movement/rotation logic must route through this.
- `tryRotate()` — rotates then attempts wall kicks at offsets `[0, -1, 1, -2, 2]` columns, keeping the first that doesn't collide.
- `clearLines()` — scans bottom-up, splices full rows out and unshifts empty rows at the top; re-checks the same row index after a splice (`r++`) since rows shift down.
- `loop(ts)` — the `requestAnimationFrame` game loop; accumulates `dt` in `dropAccum` and advances the piece (or locks it) once `dropAccum >= dropInterval`.
- `ghostY()` — projects straight down from `current` to find the landing row; used both for the ghost-piece rendering and for `hardDrop()` scoring.

### Scoring/leveling coupling

Score, lines, and level are interdependent and updated together in `clearLines()`:
- `LINE_SCORES = [0, 100, 300, 500, 800]` indexed by lines-cleared-at-once, multiplied by current `level`.
- `level = floor(lines / 10) + 1`.
- `dropInterval = max(100, 1000 - (level - 1) * 90)` ms — recalculated every time level changes.

If you change board dimensions (`COLS`, `ROWS`, `BLOCK`), the `<canvas id="board">` `width`/`height` attributes in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`) — nothing does this automatically.

## Language note

The README and code comments are in Spanish; keep new comments/docs consistent with that unless told otherwise.
