# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of standalone browser games, each a single self-contained HTML file (HTML + CSS + JS inline, no build step, no dependencies). Open any file directly in a browser to play.

**Games:**
- `tictactoe.html` — 2-player Tic Tac Toe with score tracking
- `shooter.html` — VOID SHOOTER, a retro top-down survival shooter

## Running & Testing

```bash
open tictactoe.html
open shooter.html
```

No build, no server, no install. All game state is in-memory; high scores persist via `localStorage`.

## Git Workflow

This repo is pushed to GitHub (`ezapakin-sudo/browser-games`). After every coding session, commit and push:

```bash
git add <changed files>
git commit -m "descriptive message"
git push
```

Write commit messages that describe what changed and why (e.g. `"shooter: add level 5+ scaling and strafe AI for shooter enemy"`), not just what files were touched.

## shooter.html Architecture

The game is ~400 lines of vanilla JS operating on a full-window `<canvas>`. Everything lives in module-level variables — there are no classes.

### State machine

```
S.MENU → S.PLAY → S.CLEAR (brief overlay) → S.PLAY (next level)
                ↘ S.OVER → S.MENU
```

`gs` (game state) is the single source of truth. `update()` and `draw()` both branch on it.

### Game loop

`requestAnimationFrame` with delta-time (`dt`, capped at 50 ms) drives all movement and timers. Physics values are expressed in px/s so they're frame-rate independent.

### Entity data

All entities are plain objects in module-level arrays:

| Array | Contents |
|-------|----------|
| `enemies` | Active enemy objects |
| `pBullets` | Player bullets |
| `eBullets` | Enemy bullets (shooter type only) |
| `particles` | Death-burst squares |

Entities are removed by setting `.dead = true`, then filtered out at the end of each `update()` pass.

### Enemy system

`ET` (enemy types config object) defines static properties per type: `col`, `glow`, `spd`, `hp`, `r` (radius), `pts`, `ai`, `shoots`, `shootCD`. `spawnEnemy(type)` copies these into a live enemy object and applies a per-level speed multiplier (`1 + (level-1)*0.07`).

AI behaviours are strings matched in `update()`:
- `'charge'` — beelines toward player
- `'weave'` — oscillates ±0.55 rad using `weaveT` timer (runner)
- `'strafe'` — maintains ~210 px standoff distance and circle-strafes (shooter)

### Wave system

`buildWave(lv)` returns a shuffled array of enemy type strings. Levels 1–4 use hardcoded compositions; level 5+ appends `(lv-4)` extra of each type to the level-4 base. `spawnQ` is a queue consumed by a timer (`spawnIval`, decreasing each level). The wave is complete when `spawnQ` and `enemies` are both empty.

### Rendering

All drawing is plain Canvas 2D. `ctx.shadowBlur` + `ctx.shadowColor` produce glow effects. Entities are drawn with `save/translate/rotate/restore` to avoid manual trig on every vertex. The screen-shake offset (`shakeX`, `shakeY`) is applied via a single `ctx.translate` wrapping the world draw calls.

## tictactoe.html Architecture

Simple: a 3×3 grid of `<div class="cell">` elements. Win detection checks all 8 `WINS` index triples after each move. Score state lives in the `scores` object. No canvas — all rendering is DOM + CSS.
