# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids arcade clone built with plain HTML5 Canvas and vanilla JavaScript (ES6+). No build step, no bundler, no dependencies, no package.json — the entire game logic lives in `game.js`, loaded directly by `index.html`.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js` are effective immediately on page reload.

## Architecture

Everything is in `game.js`, structured as a classic single-file game loop:

- **Entities as classes**: `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`. Entities mark themselves `dead = true` instead of removing themselves; arrays are filtered for dead entities after each update pass (in `update()`).
- **Global mutable state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` are module-level `let` bindings reassigned by `initGame()` / `nextLevel()`, not encapsulated in a class or store.
- **Game state machine**: `state` is one of `'playing' | 'dead' | 'gameover'`, checked at the top of `update(dt)` to branch behavior (respawn delay after death, restart-on-Space at game over).
- **Toroidal space**: all moving entities wrap position via the `wrap(v, max)` helper — asteroids, bullets, and the ship all re-enter from the opposite edge.
- **Asteroid splitting**: size is `1|2|3` (small/medium/large) indexing into parallel arrays `RADII`, `SPEEDS`, `POINTS`. `Asteroid.split()` produces two asteroids one size smaller; size `1` yields nothing (fully destroyed).
- **Input**: raw key state in `keys[code]`; `justPressed[code]` + `pressed(code)` gives one-shot "key down this frame" semantics (used for shooting and restart) so holding a key doesn't repeat-fire every frame.
- **Loop**: `requestAnimationFrame` drives `loop(ts)`, which computes `dt` in seconds (clamped to 0.05s max to avoid physics blow-ups on tab-switch lag) and calls `update(dt)` then `draw()`.
- **Rendering**: canvas is cleared and redrawn every frame in a fixed order — particles, asteroids, bullets, ship, then HUD/overlay — no dirty-rect optimization.

## Notes

- The README currently references power-ups and a "shooting star" asteroid type that have been removed from the game (see git history) — don't treat the README as authoritative for current features.
- UI text (HUD, game-over overlay) is in Spanish; keep new player-facing strings consistent with that.
