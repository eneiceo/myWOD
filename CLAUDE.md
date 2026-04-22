# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

MYWOD is a personal PPL (Push/Pull/Legs) training PWA. It is a **no-build, no-framework** static web app: plain HTML/CSS/vanilla JS, deployed to GitHub Pages. The entire application lives in a single file: [mywod/index.html](mywod/index.html).

## Development

No build step. Open `mywod/index.html` directly in a browser, or serve the `mywod/` folder with any static HTTP server:

```bash
# Python
python -m http.server 8080 --directory mywod

# Node (if available)
npx serve mywod
```

There are no tests, no linting setup, and no package manager.

## Deployment

Push to `main` on GitHub; GitHub Pages publishes automatically from the repo root. The app is served at `https://<user>.github.io/mywod/`. The `_config.yml` is a minimal Jekyll config required by GitHub Pages.

## Architecture

### Single-file SPA

All app code is in [mywod/index.html](mywod/index.html): embedded `<style>`, inline `<script>`, and three hidden `<div>` views toggled with `display:none/block` via `switchView()`.

### Views
- `#view-workout` — default view: block/week/day selector + exercise cards
- `#view-progress` — weight/1RM progression timelines
- `#view-info` — athlete profile and plan description

### State

All state lives in a single global object `S`, persisted to `localStorage` under key `'mywod_v5'`:

```js
S = {
  block, week, day,         // current position in the plan
  weights: {},              // working weights keyed by "block-day-exercise"
  rmHistory: {},            // 1RM test results per exercise
  weightHistory: {},        // working weight log per exercise
  openWeight, openProg,     // UI: which editor / progress card is open
  step                      // weight increment (default 2.5 kg)
}
```

`loadState()` / `saveState()` handle serialization. Every user action that changes state calls `saveState()` followed by a render function.

### Routine data

- `getSessions(block)` — returns the 6-session structure for a given block (5 training days + 1 RM test day)
- `BP` object — block parameters: intensity %, rep ranges, rest, HIIT config for each of the 3 blocks
- `LIBRARY` object — 23+ exercises with IDs, names, and Muscle & Strength URLs
- RM test days appear on weeks 6, 12, and 18 (`isRMWeek(week)`)

### Service Worker ([mywod/sw.js](mywod/sw.js))

Cache-first strategy with network fallback. Cache name is `'mywod-v2'` — **bump this version whenever cached assets change** (icons, manifest, index.html) so users get the update.

### Key functions to know

| Function | Purpose |
|---|---|
| `renderHeader()` | Redraws block tabs, week pills, day pills |
| `renderSession()` | Redraws the active session's exercise cards |
| `renderProgress()` | Redraws the full progress view |
| `getW(block, day, ex, def)` | Returns stored working weight or default |
| `setW(block, day, ex, val)` | Sets working weight and appends to `weightHistory` |
| `saveRMs()` | Reads RM input fields, stores to `rmHistory`, recalculates weights |
| `msUrl(id)` / `msName(id)` | Exercise library lookups |

### Patterns to follow

- State mutations: always call `saveState()` immediately after, then the relevant `render*()` function.
- New exercises: add to the `LIBRARY` object with a unique string ID; reference that ID in `getSessions()`.
- When changing the `localStorage` schema, bump the key from `'mywod_v5'` to avoid loading stale data, and handle migration or default initialization in `loadState()`.
- When modifying cached files, bump `CACHE_NAME` in `sw.js`.
