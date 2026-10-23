# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

No build, lint, or test tooling exists. There is no `package.json`, bundler, or test runner.

- Run: open `index.html` directly in a browser, or serve the folder with `npx serve .` and visit `http://localhost:3000`.
- Verify changes by reloading the page and playing; there are no automated checks.

## Architecture

Vanilla HTML5 Canvas game in two files:

- `index.html` holds the 800×600 `<canvas id="canvas">` and loads `game.js` as a plain classic script (not a module).
- `game.js` is the whole game, in a single `'use strict'` top-level scope. Its globals are the canvas, `ctx`, `W`/`H`, and the mutable game state (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`).

Main flow:

- **Loop:** `loop(ts)` computes `dt` (capped at 0.05s), then calls `update(dt)` and `draw()`, and reschedules via `requestAnimationFrame`. Startup is `initGame()` followed by starting the loop.
- **State machine:** `state` is one of `'playing'`, `'dead'`, or `'gameover'`. `update()` branches on it early. `'dead'` keeps asteroids and particles moving for 2s, then calls `ship.reset()`. `'gameover'` waits for Space to call `initGame()`. Changing game flow means touching this branching in `update()` and the overlay in `draw()`.
- **Input:** `keys` holds held state from `keydown`/`keyup`. `justPressed` is edge-triggered and consumed by `pressed(code)`, so use `pressed()` for one-shot actions (shooting, restart) and `keys[...]` for continuous ones (rotate, thrust). Arrow keys and Space have default prevented.
- **Collisions** are circle-based, inside `update()`: bullet vs. asteroid (`dist < a.radius`), then ship vs. asteroid with ship invincibility skipped. A destroyed asteroid calls `split()`, which spawns two smaller ones (sizes 3→2→1) and adds `POINTS[size]` to `score`.
- **Level clear:** when `asteroids` is empty, `nextLevel()` increments `level`, resets the ship, and spawns `3 + level` large asteroids.

Conventions that span the file:

- The playfield is toroidal. Every moving entity wraps with `wrap(v, max)` on `W`/`H`. Keep this in mind when adding anything that moves.
- Tuning constants (`ROT`, `THRUST`, `DRAG`, `SPEED`, `RADII`, `SPEEDS`, `POINTS`) are local to their class or update function, or top-level for asteroid tables, not in a config object.
- Each entity class has `update(dt)` and `draw()` and sets a `dead` flag that the loop filters out with `.filter(x => !x.dead)`.
- Drawing uses `ctx.save()`/`restore()` around transforms. Everything is white strokes on black, with the thruster flame in orange.
- Comments and the UI text are in Spanish (`PUNTAJE`, `NIVEL`, `ESPACIO PARA REINICIAR`). Match that when adding player-facing text.

## Suggested improvements

- `README.md` is out of date: it describes power-ups and a "estrella fugaz" (shooting star) asteroid type, which the recent commits removed from `game.js`. Remove those sections when next editing the README.
