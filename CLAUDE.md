# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A clone of the classic arcade game **Asteroids**, built with the HTML5 Canvas API and vanilla ES6+ JavaScript. No frameworks, no bundler, no dependencies, no build step, no test suite, no package.json.

The entire game lives in three files:
- `index.html` — page shell; defines the 800×600 `<canvas>` and loads `game.js`
- `game.js` — all game logic (single file, ~420 lines)
- `favicon.svg` — tab icon

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There is no build/compile/lint/test step — edit `game.js` and reload the browser to see changes.

## Architecture (`game.js`)

The whole file is organized top-to-bottom as a set of sections separated by `// ── Section ──` comment banners. Understanding the flow matters more than any single function:

1. **Input** — `keys` tracks currently-held keys; `justPressed`/`pressed()` implements edge-triggered (single-fire) presses on top of the raw keydown/keyup listeners, used for firing and restarting.
2. **Utils** — `wrap(v, max)` implements toroidal (screen-wrapping) coordinates, used by every moving entity (ship, asteroids, bullets). `dist`, `rand`, `randInt` are shared helpers.
3. **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has its own `update(dt)` and `draw()`; there is no shared base class or entity-component system. Entities mark themselves `dead = true` and are filtered out of their arrays each frame rather than removed in place.
   - `Asteroid` sizes are 1 (small) → 3 (large); `RADII`, `SPEEDS`, `POINTS` arrays are indexed by size. `split()` produces two smaller asteroids (size - 1), or nothing once size reaches 1.
   - `Ship.invincible` is a countdown timer granting temporary invulnerability (with blink effect) after spawning.
4. **Global game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) hold all mutable state. `state` is a simple string machine: `'playing' | 'dead' | 'gameover'`. `initGame()` resets everything for a new game; `nextLevel()` advances the level and respawns asteroids without resetting score/lives.
5. **`update(dt)`** — branches on `state` first. Core per-frame logic when `'playing'`: read input → move entities → collision detection (bullet↔asteroid via simple radius/distance check, splitting asteroids and spawning explosion particles; ship↔asteroid, respecting invincibility) → advance to the next level when `asteroids.length === 0`.
6. **`draw()`** — clears the canvas, draws entities in a fixed order (particles, asteroids, bullets, ship), then HUD/overlays.
7. **Main loop** — `requestAnimationFrame`-driven `loop(ts)` computes delta time `dt` (clamped to 0.05s to avoid physics blow-ups on tab-switch/lag), then calls `update(dt)` then `draw()`.

When adding new entity types or behaviors, follow the existing pattern: a class with `update(dt)`/`draw()` and a `dead` flag, pushed into one of the global arrays, filtered each frame with `array.filter(e => !e.dead)`.

Comments and UI text in this file are written in Spanish; keep that convention consistent when editing.
