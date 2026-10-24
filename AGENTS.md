# AGENTS.md

## Entrypoints
- `index.html` loads `game.js` via `<script src="game.js">`. Canvas `#canvas` (800×600). Run by opening HTML directly or via local server (see README).
- `game.js` is the single-file ES6 game (no build/bundler/deps). Everything is in global scope with classes `Bullet`, `Asteroid`, `Ship`, `Particle`.

## Commands
- Run locally: open `index.html` in browser, or `npx serve .` and visit `http://localhost:3000` (per README).
- No package scripts, tests, lint, or build configured. No lockfiles/manifest present.

## Architecture notes
- Toroidal wrap: `wrap(v,max)` used for ship/bullets/asteroids across 800×600 edges.
- Collision: circle-based. Ship-asteroid uses `dist < ship.radius + a.radius * 0.82` (forgiving). Bullet-asteroid uses `dist < a.radius`.
- Asteroid sizing: `RADII=[,16,30,50]`, `SPEEDS=[,85,55,32]`, `POINTS=[,100,50,20]` where size 1=small (100), size 2=medium (50), size 3=large (20). `split()` only if `size>1`, returns 2 asteroids of `size-1`.
- Spawning: `spawnAsteroids` avoids center (SAFE_DIST 130) around (W/2,H/2). On level up, bullets/particles cleared, ship reset, count = `3 + level` (level starts at 1).
- Invincibility: ship has 3s on reset (`invincible=3`), blinks via `Math.floor(invincible*8)%2===0`. Collisions ignored while `invincible>0`.
- State machine: `'playing' | 'dead' | 'gameover'`. On death (lives>0): `state='dead'`, 2s timer, `explode(ship.x,ship.y,14)`, `ship.dead=true`; after timer `ship.reset()` and back to `'playing'`. Game over on lives<=0; SPACE restarts via `initGame()`.
- Particles: short-lived white streaks, alpha fades by `ttl/life`.
- Input: `keys` + `justPressed` with `pressed(code)` clearing the just-pressed flag; prevents default for arrows/space.

## Gotchas
- HUD shows "NIVEL" (Spanish). Game over text in Spanish. UI is Spanish.
- Ship drawing skips if invincible blinking frame; thrust flame is random (`> 0.35`).
- `dt` capped at 0.05s in loop to avoid large jumps.
- No external assets; favicon is `favicon.svg`.
- All game state is module-global; restart is full `initGame()` (resets score/lives/level/arrays).
