# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

MYWOD is a personal training PWA for a **HYROX Open** preparation (race target: March 2027). It is a **no-build, no-framework** static web app: plain HTML/CSS/vanilla JS, deployed to GitHub Pages. The entire application lives in a single file: [index.html](index.html) at the repo root.

The plan is a 37-week / 4-phase macrocycle. Running is the limiting factor (most volume), strength is maintenance-only. Each training week has 4 session types: **A** (long Zone-2 run), **B** (strength), **C** (quality: intervals/tempo/race-pace), **D** (conditioning → compromised running → race simulation, depending on phase). The app reads the prescription per week/day, lets the user **log** what they actually did (time/distance/HR for runs, weights for strength, rounds/time for conditioning), and shows progression by phase.

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
- `#view-workout` — default view: phase/week/day selector + per-type session content + session log form
- `#view-progress` — weekly running-volume chart, pace trend, strength timelines, control tests
- `#view-info` — the "Plan" view: HR-zone settings, macro periodization, the 8 stations + substitutions, session-type glossary

### State

All state lives in a single global object `S`, persisted to `localStorage` under key `'mywod_v8'`. The previous PPL/WOD schema (`'mywod_v7'`) is a different domain and is **not** migrated — `loadState()` starts fresh.

```js
S = {
  week,                     // 1..37 (current week in the macrocycle)
  day,                      // 'A' | 'B' | 'C' | 'D' (current session type)
  hrMax,                    // user's max HR for zone calc (null until set)
  step,                     // weight increment for strength (default 2.5 kg)
  weights: {},              // strength working weights keyed by "phase-exerciseId"
  weightHistory: {},        // working-weight log per strength exercise
  logs: {},                 // session logs keyed by "week-day" (run/cond/strength entries)
  support: {},              // weekly support checklist: support[week] = {prehab, mobility, strides} counts
  openWeight, openProg      // UI: which weight editor / progress card is open
}
```

`S.support` was added additively to the same `'mywod_v8'` schema (no key bump) — older saved state without it just defaults to `{}`.

`loadState()` / `saveState()` handle serialization. Every user action that changes state calls `saveState()` followed by a render function.

### Plan data

- `PHASES` — the 4 phases (0 Base, 1 Build, 2 Específica, 3 Peak+Taper) with their week ranges, focus, and chart color. `phaseOf(n)` derives the phase from a week number.
- `WEEK_TABLE` — array of 37 entries (index 0 = week 1) holding the **variable** per-week prescription: `block` label, `deload`/`test`/`race` flags, `A` (long-run duration string), `C` (`{kind, txt}` quality session), `D` (`{kind, txt, rounds, run, stations, lines}` conditioning/sim). `getWeek(n)` augments a row with `n` + `phase`.
- `strengthDay(phase)` — the Day-B exercise list, built from a **fixed 5-pattern full-body template** (`STRENGTH_TEMPLATE`: leg / heavy pull / push / posterior chain / carry+core, pull-biased) with per-phase loading from `STRENGTH_PB[phase]`. `strengthFocus(phase)` returns the phase emphasis shown in the Day-B banner. Patterns are identical across phases, so the `phase-exerciseId` weight keys stay consistent for progression tracking.
- `SUPPORT` + `renderSupport()` / `toggleSupport()` — the weekly recurring support checklist (prehab ×2, mobility ×3, strides ×2) shown on the Hoy view, tracked per week in `S.support[week]`.
- `buildFinisher(week)` — builds the Day-D card from `D.kind` (`circuit` / `compromised` / `minirox` / `sim` / `recovery` / `race`).
- `runSession(week, dayKey)` + `C_MAP` — build the A/C run cards with target HR zone and cue.
- `STATIONS` / `CONTROL_TESTS` / `CIRCUIT` — fixed reference data (8 HYROX stations + gym substitutions, the 3 control tests, the Phase-0 technique circuit).
- `hrZones(hrMax)` — computes Z1–Z5 bpm ranges; `zoneFlag()` flags whether a logged avg HR fell in the session's target zone.
- `LIBRARY` — strength/station exercise catalog with `{url, msName, muscles}`.

### Logging

- `LOG_FIELDS` defines the input fields per session kind (`run` / `cond` / `strength`). `logForm()` renders the form (prefilled from `S.logs`), `saveLog()` writes the entry under `"week-day"`.
- Strength "logging" is the weight editors themselves (`getW`/`setW` keyed by `phase-exerciseId`), plus an RPE/note entry.

### Service Worker ([sw.js](sw.js))

Cache-first strategy with network fallback. Cache name is currently `'mywod-v8'` — **bump this version whenever cached assets change** (icons, manifest, index.html) so users get the update.

### Key functions to know

| Function | Purpose |
|---|---|
| `renderHeader()` | Redraws phase tabs, week strip (current phase, deload/test/race marks), day pills A/B/C/D |
| `renderSession()` | Dispatches by day type → `renderRun` / `renderStrength` / `renderCond`, then appends `renderSupport()` (weekly checklist) |
| `renderProgress()` | Volume chart, pace trend, strength timelines, control-test cards |
| `renderPlan()` | HR-zone settings + target paces, macro table, stations, tests, and reference cards (prevención/movilidad, técnica, pacing, fuel, autoregulation, logistics, glossary) |
| `getW(phase, id, def)` / `setW(phase, id, val)` | Read / write a strength working weight (logs to `weightHistory`) |
| `msUrl(id)` / `msName(id)` | Exercise library lookups |

### Patterns to follow

- State mutations: always call `saveState()` immediately after, then the relevant `render*()` function.
- New strength exercises: add to `LIBRARY` with a unique string ID; reference that ID in `strengthDay()`.
- To edit the plan content, edit `WEEK_TABLE` (per-week prescriptions) and/or the per-phase generators (`strengthDay`, `buildFinisher`, `C_MAP`). Keep the row shape consistent.
- When changing the `localStorage` schema, bump the key (e.g. `'mywod_v8'` → `'mywod_v9'`) and adapt `loadState()`.
- When modifying cached files, bump `CACHE` in [sw.js](sw.js).
- Charts are hand-built inline SVG (`volumeChartSvg`, `paceChartSvg`) — no chart library, to keep the no-build constraint.
