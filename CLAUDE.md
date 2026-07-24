# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clone of the classic arcade game **Asteroids**, built with plain HTML5 Canvas and vanilla JavaScript (ES6+). No dependencies, no bundler, no build step, no test suite — the entire game lives in one file, `game.js`, loaded directly by `index.html`.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build, lint, or test command — there is no `package.json`. Verify changes by opening/reloading the page in a browser and playing the game.

## Architecture

Everything is in `game.js`, organized top-to-bottom into sections (marked with `── Section ──` comments):

- **Input** — `keys` (held state) and `justPressed` (edge-triggered, consumed via `pressed(code)`) are populated by `keydown`/`keyup` listeners. `pressed()` clears the flag on read, so each key press is only consumed once per frame even though `update()` may check it once.
- **Utils** — `wrap(v, max)` implements toroidal space (objects leaving one edge reappear on the opposite edge) and is used by every moving entity's `update()`. `dist`, `rand`, `randInt` are shared helpers.
- **Entities** — `Bullet`, `Asteroid`, `Ship`, `Particle` are plain classes, each with `update(dt)` and `draw()`. None of them touch global game state directly; they only mutate their own fields. Asteroids carry a randomly generated irregular polygon (`verts`) computed once in the constructor, not regenerated per frame.
- **Game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) hold all game state. `state` is a string machine: `'playing' | 'dead' | 'gameover'`. There is no game object/class wrapping this — it's intentionally flat.
- **`update(dt)`** — branches first on `state`. Within `'playing'`, the sequence is: read input → advance all entities → filter dead entities → bullet/asteroid collisions (splits large asteroids into two smaller ones via `Asteroid.split()`, awards points via `POINTS[size]`) → ship/asteroid collision (`killShip()`) → level-clear check (`nextLevel()` when `asteroids.length === 0`).
- **`draw()`** — clears the canvas, draws entities back-to-front (particles → asteroids → bullets → ship), then HUD and any state overlay (e.g. game-over screen).
- **Main loop** — a single `requestAnimationFrame` loop (`loop(ts)`) computes `dt` in seconds (clamped to 0.05s to avoid physics jumps on tab-switch/lag) and calls `update(dt)` then `draw()`.

Canvas size is fixed at `W = 800`, `H = 600` (also set as `width`/`height` attributes in `index.html`); there is no responsive scaling.

## Conventions to follow

- Keep everything in `game.js` unless there's a strong reason to split files — the project deliberately avoids build tooling.
- New entities should follow the existing `update(dt)` / `draw()` / `dead` flag pattern so they compose with the existing filter-and-collide loop in `update()`.
- Tunable constants (speeds, radii, cooldowns, points) are declared as named `const`s near where they're used (e.g. `RADII`, `SPEEDS`, `POINTS` above `Asteroid`, or inline in `Ship.update`/`tryShoot`) — follow that pattern rather than hardcoding magic numbers inline.
- Code comments and UI text (HUD, game-over message) are in Spanish; match that when touching user-facing strings or section headers.


## Testing:
Vitest