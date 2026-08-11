# AGENTS.md

## Project

Vanilla JS Pac-Man clone (no build step, no package.json, no tests, no linter). Source of truth is `src/`; the whole game is 4 script files + canvas. Run by opening `src/index.html` in a browser, or `python3 -m http.server` from `src/`.

## Architecture

Scripts are loaded in a strict order in `src/index.html` and communicate through `window` globals — there are **no modules/imports**:

1. `js/maze.js` → defines `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`
2. `js/game.js` → state + rules; exposes `createGame`, `update`, `DIRS` (uses the maze globals)
3. `js/render.js` → canvas drawing; uses `DIRS`; exposes `draw`
4. `js/main.js` → game loop, keyboard, overlay screens

To add a JS file you must add a `<script>` tag manually in the right position — the game breaks silently if a dependency loads late.

Key gotchas:

- `MAZE` is the **pristine** maze; each `createGame()` copies it into `game.grid`. Never mutate `MAZE` — read/mutate `game.grid` (it tracks eaten dots). Don't reset the game by editing `MAZE`.
- The maze is authored as ASCII strings in `maze.js`. Tile codes: `#`=wall(1), `.`=dot(2), `-`=door/pen gate(3), space=empty(0). Coordinates are (x, y), origin top-left, maze is 28x31.
- Movement is grid-aligned: speeds are fractions of a cell per frame (`PACMAN_SPEED = 0.125`, `GHOST_SPEED = 0.1`) and turns only apply when the actor is `aligned()` to a cell center.
- Ghosts: `kind: 'hunter'` chases Pac-Man greedily, `kind: 'random'` picks randomly. The two ghosts start inside the pen behind the door.
- `render.js`'s `TILE = 20` must stay consistent with the 560x620 canvas (28x31 cells) in `index.html`.

## Conventions

- Comments, code identifiers, and README are **Spanish** — write new comments and specs in Spanish (specs must match existing ones).
- Code style: spaces inside parentheses `( x )`, single quotes, semicolons, `const` first. Match the existing style in edited files.
- The game loop is a plain `requestAnimationFrame`; the `frame` counter drives Pac-Man's animated mouth.

## Workflow: spec-driven development

This repo exists to practice the spec-driven method, enforced via two installed skills (`.agents/skills/`):

- Start large features with the `/spec` skill → writes into `specs/<NN-name>.md` following `template.md`.
- Implement an approved spec with `/spec-impl` → validates the spec state is "Approved", creates a git branch named after the spec, and implements step-by-step with pauses to review diffs.
- Use `AskUserQuestion` for clarifications, and reply in the language of the user's prompt.

`skills-lock.json` pins both skills to `klerith/fernando-skills`.
