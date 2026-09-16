# Ability Draft Assistant

> Real-time pick recommender for Dota 2 **Ability Draft** — reads the draft board from your screen, scores every available ability, and highlights the best picks in a desktop overlay.

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Electron](https://img.shields.io/badge/Electron-47848F?logo=electron&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?logo=react&logoColor=61DAFB)
![SQLite](https://img.shields.io/badge/SQLite-003B57?logo=sqlite&logoColor=white)
![Tests](https://img.shields.io/badge/tests-pytest%20%7C%20vitest-brightgreen)

<!-- TODO: add a demo GIF here — this is the single most valuable thing on the page -->

## How it works

```
 ┌──────────────── offline ────────────────┐   ┌──────────────────── runtime ─────────────────────┐
 Windrun API ─► pipeline (Python)             │   screen capture ─► frame-diff gate
   httpx client · transform · SQLite loader   │        │
   icon assets ─► perceptual-hash index ──────┼──►  recognition (pHash nearest-match per grid cell)
 └────────────────────────────────────────────┘        │
                                                   scoring kernel: hard filter ─► dynamic multi-signal score ─► Top-K
                                                       │
                                                   IPC snapshot ─► Electron overlay (React + Zustand)
```

| Package | Role |
|---|---|
| `pipeline/` | Python ETL: fetches hero/ability win-rate and pair data, normalizes IDs, loads into SQLite, builds a perceptual-hash icon index |
| `core/` | Recognition (pHash matching, ROI layout derived from window size) and the scoring kernel (win-rate, synergy pairs, Aghanim's, pick position signals); offline backtest of Top-K hit rate |
| `main/` | Screen capture loop with frame-diff gating and snapshot building |
| `renderer/` | Overlay view-model, stale-frame guard, live weight-tuning control panel |
| `app/` | Electron shell: main process, `contextBridge` preload, typed IPC |
| `shared/` | Types shared across packages |

## Engineering highlights

- **Two-stage architecture**: heavy data work runs offline; the runtime path only does hashing, lookups, and scoring.
- **Pluggable scoring signals** with configurable weights and an offline backtest harness to compare configurations.
- **Resolution-independent recognition**: board regions derived from the window rect instead of hard-coded 1080p coordinates.
- **Test-driven**: pytest (with HTTP mocking) for the pipeline, vitest for every TS package, including performance-budget assertions on the frame-diff gate.

## Getting started

```bash
# 1. build the data + icon index
cd pipeline && pip install -e ".[dev]" && python -m ad_pipeline --help

# 2. install JS workspace and run the overlay (mock data source)
cd .. && npm install
npm run dev --workspace app
```
<!-- TODO: verify these commands match your scripts -->

## Roadmap

- [x] Offline data pipeline & icon index
- [x] Recognition + scoring kernel with backtest
- [x] Overlay logic layer and mock-driven Electron shell
- [ ] Wire live capture into the Electron app
- [ ] Demo video & packaged release
