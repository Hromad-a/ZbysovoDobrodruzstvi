# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Zbýšovo dobrodružství" (working title "Kočka na vodítku") — a cozy top-down walking game: walk a black cat on a leash through a Prague-style panelák neighbourhood, meet philosophizing hedgehogs, reach the person waiting on the other side. All in-game text and dialogue is in Czech.

## Running

The whole game is a single `index.html` with inline CSS and JS — no build step, no dependencies to install, no tests. Rough.js is loaded from a CDN, so an internet connection is required.

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
- **Entities & game state** — `gameState` flows `start → play → ending → done`. The cat is a small FSM: `wander` (orbits the player with sniff pauses) → `approach` → `lay` (scripted hedgehog conversation via a timed `cat.script` array) → back to `wander`; the ending uses `toDest → petted`. The leash is a hard distance constraint clamping the cat to `LEASH_LEN` from the player.
- **Input** — merged keyboard + touch joystick through `inputVec()`.
- **Update / render loop** — fixed logic in `update(dt)` (dt clamped to 0.05), rendering in `render(now)`: world blit → shadows → leash → y-sorted entities (`ents.sort` by y, painter's algorithm) → bubbles → HUD arrow → joystick → paper-grain overlay.

## Conventions

- Keep the hand-drawn aesthetic: any new visual element should be drawn through Rough.js with the shared `opts()` helper, use the `PAL` palette, get 2 seed variants for the boiling-line effect, and use deterministic seeds/jitter (no per-frame randomness in art).
- New world obstacles need a matching `block()` collider.
- New animated characters follow the existing pattern: a `draw<Name>(rc, g, seed, pose)` function, pose variants registered in `SPR`, anchor at bottom-center.
- Player-visible text stays in Czech.
