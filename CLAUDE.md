# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Zbýšovo Dobrodružství" — a cozy top-down walking game: walk a black cat on a leash through a Prague-style panelák neighbourhood, meet philosophizing hedgehogs, and reach the person who arrives by tram (2–5 minutes into the game) on the other side. All in-game text and dialogue is in Czech.

## Running

The whole game is `index.html` with inline CSS and JS — no build step, no dependencies to install, no tests. The only dependencies are vendored locally so the game works offline and from `file://` (do not swap them for CDN links): Rough.js 4.6.6 as `rough.min.js`, and the Pangolin font (`Pangolin-Regular.ttf`, OFL-licensed, handwriting style with Czech diacritics) as the primary game font on all platforms.

```
python3 -m http.server 8000
```

or just open `index.html` in a browser. Works on desktop (WASD / arrows) and mobile (floating touch joystick via pointer events).

## Architecture

Everything lives in the single `<script>` block of `index.html`, organized into banner-commented sections in this order:

- **Constants / palette** — `INK`, `PAPER`, `PAL` color palette, world size (`WORLD_W`/`WORLD_H` = 2000×1500), `LEASH_LEN`, `BOIL_MS`.
- **Sprites** — all art is procedurally drawn with Rough.js, in a deliberate sketchy child-drawing / Die Brücke style (this is intentional placeholder art, to be swapped for hand-drawn PNGs later). `makeSprite()` pre-renders each sprite to 2 offscreen canvases with different Rough.js seeds; the renderer alternates between them every `BOIL_MS` for a "boiling line" animation. Rough.js options come from the `opts(seed, fill, style, extra)` helper — seeds are explicit and deterministic so the art is stable frame to frame.
- **World** — static layout data (`BUILDINGS`, `CARS`, `BINS`, `TREES`, roads, park polygon). Obstacles register AABB colliders via `block()`. The whole world is pre-rendered once into 2 boil-variant canvases (`worldLayers`); the camera just blits a viewport from them.
- **Dialogue** — string arrays (`HEDGE_LINES`, `HEDGE_ALT`, `CAT_LINES`), Czech. Speech bubbles go through `say(ent, text, dur)`.
- **Entities & game state** — `gameState` flows `start → play → ending → done`. The cat is a small FSM: `wander` (orbits the player with sniff pauses), `approach → lay` (scripted hedgehog conversation via a timed `cat.script` array; delivering a carried worm swaps in a thank-you script with useful hints), `toWorm → playWorm` (bats a park earthworm around, then carries it; ~35% of the time he later eats it in `eatWorm` and apologizes instead of delivering), `dashTree → inTree` (climbs a nearby tree for ~3s, drawn via `cat.zOff` height offset), and `toDest → petted` for the ending. The leash is a hard distance constraint clamping the cat to `LEASH_LEN` from the player. The destination person starts `away` and arrives after a 2–5 min timer via one of two per-game random modes (`arrivalMode`): by tram (`tram` entity), or by coming down from the north-east panelák after occasionally peeking from its window (`peek`), then `walking → waiting`; the ending requires `waiting`. Worms spawn only inside the `PARK` polygon (`inPoly` test). A day-night cycle driven by `playT` (`dayPhase()`, full dark ~5 min) tints the scene toward darkness in `drawNight()`, punching light pools out of the darkness overlay for street `LAMPS` (staggered switch-on with flicker) and lit windows (`LITWINS`).
- **Input** — merged keyboard + touch joystick through `inputVec()`.
- **Update / render loop** — fixed logic in `update(dt)` (dt clamped to 0.05), rendering in `render(now)`: world blit → shadows → leash → y-sorted entities (`ents.sort` by y, painter's algorithm) → bubbles → HUD arrow → joystick → paper-grain overlay.

## Conventions

- Keep the hand-drawn aesthetic: any new visual element should be drawn through Rough.js with the shared `opts()` helper, use the `PAL` palette, get 2 seed variants for the boiling-line effect, and use deterministic seeds/jitter (no per-frame randomness in art).
- New world obstacles need a matching `block()` collider.
- New animated characters follow the existing pattern: a `draw<Name>(rc, g, seed, pose)` function, pose variants registered in `SPR`, anchor at bottom-center.
- Player-visible text stays in Czech. The cat is male (a kocour) — keep his dialogue and any text about him in masculine gender agreement.
