# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

MYWOD is a personal training PWA. It is a **no-build, no-framework** static web app: plain HTML/CSS/vanilla JS, deployed to GitHub Pages. The entire application lives in a single file: [index.html](index.html) at the repo root.

## Development

No build step. Open [index.html](index.html) directly in a browser, or serve the repo root with any static HTTP server:

```bash
# Python
python -m http.server 8080

# Node (if available)
npx serve .
```

There are no tests, no linting setup, and no package manager.

## Deployment

Push to `main` on GitHub; GitHub Pages publishes automatically from the repo root. The app is served at `https://<user>.github.io/mywod/`. The [_config.yml](_config.yml) is a minimal Jekyll config required by GitHub Pages.

## Architecture

### Single-file SPA

All app code is in [index.html](index.html): embedded `<style>`, inline `<script>`, and three hidden `<div>` views toggled with `display:none/block` via `switchView()`.

### Views
- `#view-workout` — default view: block/week/day selector + exercise cards + finisher block
- `#view-progress` — weight progression timelines + stats bar
- `#view-info` — athlete profile and plan description

### State

All state lives in a single global object `S`, persisted to `localStorage` under key `'mywod_v7'` (with automatic migration from legacy `'mywod_v6'` and `'mywod_v5'` keys — old `rmHistory` and `attendanceLog` fields are dropped):

```js
S = {
  block, week, day,         // current position in the plan
  weights: {},              // working weights keyed by "block-day-exercise"
  weightHistory: {},        // working weight log per exercise (manual edits)
  openWeight, openProg,     // UI: which editor / progress card is open
  step                      // weight increment (default 2.5 kg)
}
```

`loadState()` / `saveState()` handle serialization. Every user action that changes state calls `saveState()` followed by a render function.

### Routine data

- 4 sessions per block: **Push / Pull / Legs / WOD**. No RM test day.
- `getSessions(block)` — returns the 4-session structure for a given block.
- `BP` object — block parameters: rep range, rest, label, set counts, core format (EMOM/AMRAP/For Time). Blocks are 6 weeks each, 3 blocks total (18 weeks).
- `coreBlock(block, sessionIdx)` — generates the daily core+conditioning finisher for push/pull/legs sessions (rotates exercises, format follows the block).
- `wodExercises(block)` / `wodMetcon(block)` — content of the WOD day: olympic lifts + crossfit compounds + structured metcon.
- `LIBRARY` object — exercise catalog with `{url, msName, muscles}`. Includes PPL classics + WOD movements (power-clean, snatch, jerk, thruster, wall-ball, kb-swing, box-jump, burpee, etc).

### Service Worker ([sw.js](sw.js))

Cache-first strategy with network fallback. Cache name is currently `'mywod-v4'` — **bump this version whenever cached assets change** (icons, manifest, index.html) so users get the update.

### Key functions to know

| Function | Purpose |
|---|---|
| `renderHeader()` | Redraws block tabs, week pills, day pills |
| `renderSession()` | Redraws the active session: exercise cards + core/metcon finisher |
| `renderProgress()` | Redraws the full progress view with timelines |
| `getW(block, day, ex, def)` | Returns stored working weight or default |
| `setW(block, day, ex, val)` | Sets working weight and appends to `weightHistory` |
| `msUrl(id)` / `msName(id)` | Exercise library lookups |

### Patterns to follow

- State mutations: always call `saveState()` immediately after, then the relevant `render*()` function.
- New exercises: add to the `LIBRARY` object with a unique string ID; reference that ID in `getSessions()` / `wodExercises()`.
- When changing the `localStorage` schema, bump the key (e.g. `'mywod_v7'` → `'mywod_v8'`) and add a fallback `localStorage.getItem` chain in `loadState()` that reads the previous key, then `removeItem` the old one after the first successful save.
- When modifying cached files, bump `CACHE` in [sw.js](sw.js).
- Pesos: ya no se autocalculan por % de 1RM. Cada ejercicio define un `weight` literal como sugerencia inicial; el usuario lo ajusta a mano y queda persistido en `S.weights` / `S.weightHistory`.
