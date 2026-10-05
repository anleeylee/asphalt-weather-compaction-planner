# Asphalt Weather & Compaction Planner

**Model-based estimate of the HMA cooling/compaction window for asphalt paving planning** — a documented lumped thermal model, not a guess.

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#usage)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**English** | [简体中文](./README.zh-CN.md)

---

## What it solves

Hot-mix asphalt (HMA) has a limited window between delivery and final compaction — air temperature, wind, base temperature and lift thickness all shorten or extend it, and getting it wrong means failed compaction and rework. This tool runs a **documented, versioned lumped thermal model (ADI-THM-v1.0)** over air temp, wind, surface/base temp, lift thickness and delivery temperature to estimate the **start-roll / stop-roll window and cooling curve**.

The AI explains model outputs but never replaces the thermal model, and the tool never says "approved to pave" — supplier/agency limits override planning defaults.

## Features

- **HMA cooling/compaction window** — start-roll / stop-roll estimates from a lumped thermal model
- **Documented model versioning** — ADI-THM-v1.0, reproducible every run
- **Inputs** — air temp, wind, surface/base temp, lift thickness, delivery temperature
- **Cooling curve output** — temperature over time for paving planning
- **Honest limits** — never "approved to pave"; supplier/agency limits take precedence
- **AI boundary** — AI explains, the thermal model decides

## Install

```bash
pip install -r requirements.txt
```

## Usage

```bash
python -m scripts.s08_weather_compaction_planner --project ./project --air-temp 60 --wind 5 --lift-in 3
```

Run with `--help` for all options. Every script in this family shares the same CLI convention: `--project <path> --input <path> --output <path> --format json|csv|md|xlsx|pdf --config <path> --verbose --dry-run`.

## Outputs

- `thermal_run.json` — inputs snapshot, model version, window, assumptions, limitations, risk factors
- `thermal_summary.md` — planning summary

## How it works

- **Standard lifecycle** — `DISCOVER → INGEST → EXTRACT → NORMALIZE → VALIDATE → CALCULATE → OUTPUT → AUDIT`
- **Deterministic thermal model** — documented lumped model, versioned and reproducible
- **AI explains, never replaces** — model outputs drive the window, not the AI

## Quality gates

- deterministic schema validation on every record;
- golden sample fixtures with direction checks (`tests/`);
- review queue: low-confidence values, ambiguous scales and conflicts land in `review_queue.json` / summary — never a silent fix;
- audit log: every run writes `run_id`, timings, input hashes, engine version and outputs to `<project>/audit/`.

## Testing

```bash
python -m pytest -q
```

## Security & privacy

Local files stay local by default. API keys live in environment variables (`ASPHALTCOSTS_API_KEY`, `ADI_AI_API_KEY`) — never in source code. Derived files are written to `working/` or `output/`; source files are never modified.

## License

MIT — see [LICENSE](LICENSE). Part of the [Asphalt Desktop Intelligence](https://github.com/anleeylee/asphalt-desktop-intelligence) toolkit.

## The calculation engine

[**AsphaltCosts.com**](https://asphaltcosts.com/) is the deterministic calculation layer: area → compacted volume → net tons → order tons (allowance applied once) → truckloads → material cost, with sourced planning defaults (145 lb/ft³ FHWA density, editable allowance and truck capacity). This tool feeds measured and validated inputs into that engine (or its labeled local mirror, `asphaltcosts-web-engine/1.0-mirror`) and never re-implements the formulas.
