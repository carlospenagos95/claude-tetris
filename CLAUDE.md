# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris implementation. No build step, no dependencies, no package.json. Three files: `index.html` (DOM/canvas structure), `style.css` (dark/retro theme), `game.js` (all game logic, ~300 lines).

## Running

Open `index.html` directly in a browser, or serve statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

No test suite, linter, or build tooling exists in this repo.

## Architecture

Everything lives in `game.js` as module-level state and functions (no classes, no framework):

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: `PIECES` are square matrices; rotation is done by `rotateCW` (transpose + reverse), not by precomputed rotation states.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells — used for movement, rotation, and ghost-piece projection.
- **Wall kicks**: `tryRotate()` rotates then retries at x-offsets `[0, -1, 1, -2, 2]` until one doesn't collide.
- **Game loop**: `requestAnimationFrame`-driven `loop(ts)` accumulates delta time in `dropAccum`; when it exceeds `dropInterval`, the piece drops a row or locks (`lockPiece` → `merge` + `clearLines` + `spawn`).
- **Line clearing**: `clearLines()` scans bottom-up, splices full rows out and unshifts empty rows in; re-checks the same index (`r++`) after a splice since rows shift down.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` × `level`; level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down via repeated `collide` checks; drawn at `globalAlpha = 0.2`.
- **Rendering**: single `draw()` per frame redraws grid, locked board, ghost, and current piece onto `#board`; `drawNext()` renders the preview piece onto a separate `#next-canvas`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

## Controls

Arrow keys move/rotate/soft-drop, Space hard-drops, P pauses. Input handling is a single `keydown` listener with a `switch` on `e.code`; pause/game-over states short-circuit it.
