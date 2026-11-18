# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

MYWOD is a personal **gym training PWA**. It is a **no-build, no-framework** static web app: plain HTML/CSS/vanilla JS, deployed to GitHub Pages. The entire application lives in a single file: [index.html](index.html) at the repo root.

The program is organized **by months**. Each month is 4 weeks with a fixed weekly structure of **3 gym days + 2 early-morning calisthenics days + 1 optional extra** (the roles never change; the content inside them rotates every week). `DAY_META[k].dow` records the weekday each role falls on:

- **Día A** — Lunes. Piernas pesado (Sentadilla + Peso muerto, both as `principal`)
- **Día B** — Miércoles. Tren superior (empuje/tracción) + correr
- **Día C** — Viernes. Potencia + cuerpo completo (Clean and jerk + Snatch)
- **Día D** — Fin de semana, optional. Alternates by week: benchmark (W1, W3), accessories/isolation (W2), easy aerobic (W4).
- **Día E** — Martes. Calistenia: control y zona media.
- **Día F** — Jueves. Calistenia: movilidad y piernas.

On load the app opens on the day matching today's weekday (via `diaDeHoy()`, derived from `DAY_META.dow`) and marks that pill "hoy"; month and week come from saved state.

The two calisthenics days are done **at home with nothing but a mat** (floor + a wall are the only other requirements — no bar, parallettes, box or stick), 30-40 min, deliberately **low fatigue** (técnica → zona media → movilidad, never to failure): they sit between gym days, so they must not steal recovery. Day E follows heavy legs, so it loads neither the lower back nor the legs and keeps the upper body fresh for Wednesday; Day F follows the upper-body day and prepares ankle, hip and shoulder for Friday's olympic lifts. They back off in week 4 like everything else. They carry `ownWarmup:true` (the standard barbell warm-up does not apply) and their own `note` in `DAY_META` (the week's theme talks about bar loads, which would be wrong on these days).

Each training day (A–C) follows a fixed order: **entrada en calor** (a constant warm-up) → **fuerza principal** (heavy lift, 2–3 min rest; Day A has two) → **fuerza secundaria** (1–2 min rest) → **superserie** → **aislados** (single-joint work that the three compound days would otherwise miss) → **zona media** (core rounds) → **circuito** (or a run on Day B). The warm-up renders on every session except pure-recovery days (those whose only block is a `CONTROL` circuit).

Four months are loaded: **Mes 1** (Base → Más pesado/menos reps → Más volumen → Descarga), **Mes 2** (Carga → Fuerza máxima → Volumen alto → Descarga), **Mes 3** (Acumulación → Intensificación → Pico → Test y descarga) and **Mes 4** (Acumulación → Más carga → Volumen máximo → Descarga), each 4 weeks. **Mes 4 is a hypertrophy block** (8–12 reps, 1–2 in reserve) and the first month with **suggested kg on every exercise**, derived from the user's working weights at the end of Mes 3 (squat 115, deadlift 100, military press 55, C&J 65, snatch 55, bench 60, barbell row 60…): main lifts at ~65–75 % of the estimated max; accessories at the user's weight in W1, one step up with fewer reps in W2, one step down with more reps in W3, lighter in W4. Olympic lifts stay technical (no load increase) during hypertrophy. Mes 3 peaks the block: week 4 is a short **test week** where Days A and B work up to a heavy triple on squat, deadlift and military press plus a max set of pull-ups, with accessories cut. The olympic lifts are deliberately not tested. Across months the four main lifts stay fixed (Sentadilla, Press militar, Clean and jerk, Peso muerto) so progress stays measurable, while secondary lifts, accessories and circuits rotate. "Cindy" is re-tested on Week 1 / Day D of every month as the benchmark. The app is read-only guidance: it shows the prescription per month/week/day and remembers your position; it does not log workouts.

**Wrist constraint (from Mes 4):** the wrists suffer under load, so the calisthenics days avoid every hand-supported position: no handstand, L-sit, wall walk, shoulder-tap plank or all-fours drills (the shared mobility uses *Libro abierto* and *Rodillas al pecho* instead). Day E is classic core (built with `diaCore`), Day F is push-ups **on the fists** + *espinales* plus core (built with `diaFlex`). Planks go on the forearms. Months 1-3 keep their original `diaE`/`diaF` sessions as history.

**Injury constraint (lower back):** the contraindicated movements are the **incline press**, the **leg press**, and the whole **hip-thrust family** — anything that drives the hips up from a supine position, which includes the barbell hip thrust and the glute bridge. Keep them out of every month, not just the newest one. Lateral raises and dips ARE allowed. The floor press covers the incline press slot; glute work goes through movements that keep the torso upright (sentadilla búlgara, zancadas inversas, subidas al cajón).

## Development

No build step. Open [index.html](index.html) directly in a browser, or serve the repo root with any static HTTP server:

```bash
python -m http.server 8080
```

There are no tests, no linting setup, and no package manager.

## Deployment

Push to `main` on GitHub; GitHub Pages publishes automatically from the repo root. The app is served at `https://<user>.github.io/mywod/`. The [_config.yml](_config.yml) is a minimal Jekyll config required by GitHub Pages.

## Architecture

### Single-file SPA

All app code is in [index.html](index.html): embedded `<style>`, inline `<script>`, and two hidden `<div>` views toggled with `.active` via `switchView()`.

- `#view-workout` — month tabs + week strip + day pills (A–F), then the session for the selected month/week/day.
- `#view-info` — the "Referencia" view: notation, circuit formats, weekly structure, exercise glossary, progress tracking.

### State

Only the user's position, persisted to `localStorage` under `'mywod_v9'`:

```js
S = { month, week, day }  // day: 'A'..'F'
```

`loadState()` / `saveState()` validate against `PROGRAM`. Every state change calls `saveState()` then the relevant render function.

### Plan data

- `PROGRAM.months` — array of `{n, name, weeks}`; each week is `{n, theme, note, deload?, days}`; `days` maps A–D to an ordered array of **blocks**.
- **Block constructors:**
  - `heavy(name, scheme, role, note, kg)` — strength lift. `role` `'principal'` (rest 2–3 min) or `'secundaria'` (1–2 min); rest is derived at render time. `scheme` is a free string (`'5×3'`, `'4 × máx'`, `'hasta 1×3 pesado'`). `kg` is optional, a number (`87.5`), rendered as a "87,5 kg" chip. In supersets/accessories the load goes inside the move string: `'Curl femoral — 12 · 55 kg'` (dumbbell loads are per hand).
  - `ss(rounds, moves, label)` — superset.
  - `iso(rounds, moves)` — isolation work ("Aislados").
  - `core(rounds, moves)` — zona media.
  - `skill(rounds, moves)` — calisthenics skill practice ("Técnica", never to failure).
  - `mob(moves, label)` — one pass of mobility drills; `label` names the section.
  - `run(presc, cue)` — running prescription.
  - `circ({fmt, dur, title, scheme, steps, note, name})` — circuit; `fmt` keys into `FMT`.
- `CALISTENIA[month][week] = {E, F}` — built by `diaE`/`diaF` from the shared mobility constants (`MOB_E_IN`, `MOB_E_OUT`, `MOB_F_IN`, `MOB_F_OUT`) and merged into `PROGRAM` by the loop right after it.
- `WARMUP` — barbell warm-up; skipped on pure-recovery days and on days flagged `ownWarmup`.
- `FMT` — `FOR_TIME`, `AMRAP`, `EMOM`, `CHIPPER`, `CONTROL`.
- `DAY_META` — role, color `type`, badge, `dow`, title, optional `note`/`optional`/`ownWarmup`. `DAY_ORDER` is `['A','B','C','D','E','F']`. `ACCENT` maps `type` → hex.
- `GLOSARIO` — `[{cat, items:[{name, desc}]}]`. It must match the routine exactly: every movement used has an entry, and no entry is left unused.

### Color coding (by day)

`A` strength/yellow, `B` aerobic/blue, `C` quality/red, `D` surprise/green, `E` cond/purple, `F` calis/teal.

### Service Worker ([sw.js](sw.js))

Cache-first with network fallback. Cache name is currently `'mywod-v23'` — **bump it whenever cached assets change**.

## Patterns to follow

- **Add a new month:** append `{n, name, weeks:[...]}` to `PROGRAM.months` (mind the comma after the previous month) **and** add `CALISTENIA[n]` for weeks 1-4, or days E/F will be missing.
- **Edit prescriptions** with the block constructors, keeping the per-day order (principal → secundaria → superserie → aislados → zona media → circuito/run).
- **Replacing an exercise across the program:** edit the specific blocks, never pair old/new lines by position from a diff — neighbouring changed lines get mismatched and a superset can silently disappear. After any bulk edit, check that every A–C day (except the Mes 3 test week) still has exactly one `super` and at most one `iso`.
- State mutations: `saveState()` then `render*()`.
- Changing the `localStorage` schema: bump the key and adapt `loadState()`.
- No frameworks, no build step.
