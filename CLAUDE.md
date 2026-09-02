# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

MYWOD is a personal **gym training PWA**. It is a **no-build, no-framework** static web app: plain HTML/CSS/vanilla JS, deployed to GitHub Pages. The entire application lives in a single file: [index.html](index.html) at the repo root.

The program is organized **by months**. Each month is 4 weeks with a fixed weekly structure of **5 day roles** (the roles never change; the content inside them rotates every week):

- **Día A** — Piernas pesado + circuito corto
- **Día B** — Tren superior + correr
- **Día C** — Potencia + saltos
- **Día D** — Cuerpo completo + circuito largo
- **Día E** — Día sorpresa (opcional)

Each day (A–D) follows a fixed order: **entrada en calor** (a constant warm-up) → **fuerza principal** (heavy lift, 2–3 min rest) → **fuerza secundaria** (secondary lift, 1–2 min rest) → **superserie(s)** (one or two) → **zona media** (core rounds) → **circuito** (or a run on Day B). Day E is a single standalone WOD (no warm-up section). Today only **Mes 1** is loaded (4 weeks: Base → Más pesado/menos reps → Más volumen → Descarga), but the data structure is built to add more months later. The app is read-only guidance: it shows the prescription per month/week/day and remembers your position; it does not log workouts.

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

All app code is in [index.html](index.html): embedded `<style>`, inline `<script>`, and two hidden `<div>` views toggled with `.active` (display none/block) via `switchView()`.

### Views
- `#view-workout` — default view: month tabs + week strip + day pills (A–E), then the session content (strength lifts, supersets, circuits, runs) for the selected month/week/day.
- `#view-info` — the "Referencia" view: how to read notation (series×reps, superserie, descanso), the circuit-format glossary, the weekly structure, the exercise glossary, and how to measure progress.

### State

State is minimal — just the user's position in the program — persisted to `localStorage` under key `'mywod_v9'`. The previous HYROX schema (`'mywod_v8'`) is a different domain and is **not** migrated.

```js
S = { month, week, day }  // month: 1-based month number; week: week number within the month; day: 'A'|'B'|'C'|'D'|'E'
```

`loadState()` / `saveState()` handle serialization and validate that `month`/`week`/`day` exist in `PROGRAM`. Every user action that changes state calls `saveState()` then the relevant render function.

### Plan data

- `PROGRAM` — the whole plan: `PROGRAM.months` is an array of months; each month has `{n, name, weeks}`; each week has `{n, theme, note, deload?, days}`; `days` maps `A`–`E` to an ordered array of **blocks**. `currentMonth()` / `currentWeek()` resolve the selected position.
- **Block constructors** keep the data compact:
  - `heavy(name, scheme, role, note)` → `{k:'heavy', ...}` — a strength lift. `role` is `'principal'` (rest 2–3 min) or `'secundaria'` (rest 1–2 min); rest is derived from `role` at render time, not stored. `scheme` is a free string (`'5×3'`, `'4 × máx'`).
  - `ss(rounds, moves, label)` → `{k:'super', ...}` — a superset (N rounds, list of move strings, optional `label` like `'Superserie 1'`).
  - `core(rounds, moves)` → `{k:'core', ...}` — a zona-media (core) block; same shape as a superset, rendered with the "Zona media" label.
  - `run(presc, cue)` → `{k:'run', ...}` — a running prescription.
  - `circ({fmt, dur, title, scheme, steps, note, name})` → `{k:'circ', ...}` — a circuit. `fmt` keys into `FMT`.
- `WARMUP` — the constant `{dur, steps}` warm-up rendered at the top of every non-optional day (A–D).
- `FMT` — circuit-format metadata: `FOR_TIME`, `AMRAP`, `EMOM`, `CHIPPER`, `CONTROL`, each `{name, tag}`.
- `DAY_META` — the fixed role, color `type`, badge, and title for each day A–E; `DAY_ORDER` is `['A','B','C','D','E']`. `ACCENT` maps each day `type` to its hex color (used for inline accent styling of superset/core/circuit cards).
- `GLOSARIO` — the exercise catalog, grouped by category: an array of `{cat, items:[{name, desc}]}` shown in the Referencia view.

### Rendering

- `renderHeader()` — draws month tabs (from `PROGRAM.months`, plus a locked "próximo" placeholder), the week strip (deload weeks marked `↓`), and the A–E day pills.
- `renderSession()` — reads the selected week/day, sets the title/badge/banner, renders the warm-up (for non-optional days), then iterates the day's blocks with a running number. It prepends a section label when the block **group** changes (`groupOf()` → Fuerza principal / Fuerza secundaria / Superseries / Zona media / Correr / Circuito) and dispatches to `renderHeavy` / `renderSuper` (also handles `core`) / `renderRun` / `renderCirc`.
- `renderRef()` — builds the Referencia cards (cómo leer, formatos de circuito, estructura de la semana, glosario, progreso).

### Color coding (by day)

Each day role has a color, applied via CSS classes and JS `ACCENT` hexes:
`A` strength/yellow, `B` aerobic/blue, `C` quality/red, `D` cond/purple, `E` surprise/green.

### Service Worker ([sw.js](sw.js))

Cache-first strategy with network fallback. Cache name is currently `'mywod-v12'` — **bump this version whenever cached assets change** (icons, manifest, index.html) so users get the update.

## Patterns to follow

- **Add a new month:** push a `{n, name, weeks:[...]}` object onto `PROGRAM.months`. The month tabs render automatically. Keep the week/day/block shapes consistent.
- **Edit prescriptions:** edit the relevant `days` arrays using the block constructors (`heavy`, `ss`, `core`, `run`, `circ`). Keep the per-day order (principal → secundaria → superserie(s) → zona media → circuito/run). Don't hand-write block objects unless adding a new field.
- **New exercise explanation:** add a `{name, desc}` entry under the right category in `GLOSARIO`.
- **New circuit format:** add an entry to `FMT` and reference its key as `fmt` in a `circ(...)` block.
- State mutations: always `saveState()` immediately after, then the relevant `render*()`.
- When changing the `localStorage` schema, bump the key (e.g. `'mywod_v9'` → `'mywod_v10'`) and adapt `loadState()`.
- When modifying cached files, bump `CACHE` in [sw.js](sw.js).
- No frameworks, no chart libraries, no build step — keep the no-build constraint.
