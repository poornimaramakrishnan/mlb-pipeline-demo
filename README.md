# ⚾ MLB Data Pipeline — Live Demo

> **Real-time MLB statistics pipeline** — scrapes the official MLB Stats API
> and transforms data into production-ready formats.
> Click the button below to run it yourself.

[![Run Demo](https://img.shields.io/badge/▶_Run_Demo-Click_Here-blue?style=for-the-badge&logo=github-actions)](../../actions/workflows/demo.yml)
[![Docker Image](https://img.shields.io/badge/Docker-GHCR-2496ED?style=flat-square&logo=docker)](https://ghcr.io/poornimaramakrishnan/mlb-pipeline)

---

## What This Pipeline Does

| Phase | Description | Output |
|-------|-------------|--------|
| **Phase 1** | Scrapes MLB Stats API → 57-column CSV box scores | `.csv` files |
| **Phase 2** | Converts game data → structured `.dat` text (Retrosheet-style) | `.dat` files |
| **Phase 3** | SQL database ingestion + reporting *(Milestone 2 — coming soon)* | — |

### Data Columns (Phase 1 CSV — 57 columns)

Game metadata, team info, scores by inning, pitching stats (W/L/S, ERA, K, BB),
batting stats (H, HR, RBI, AVG), fielding stats, and more.

### Structured Text (Phase 2 .dat — 15 sections)

Game header, venue/weather, line scores, pitching lines, batting lines,
fielding summaries, play-by-play highlights, and game notes — all in a
human-readable fixed-width format suitable for legacy system ingestion.

---

## ▶️  How to Run the Demo

1. Click the **[Run Demo](../../actions/workflows/demo.yml)** badge above
   (or go to the **Actions** tab → **"Run MLB Pipeline Demo"**).
2. Click the green **"Run workflow"** button.
3. Configure your run using the inputs below:

### ☑️ Phase toggles (checkboxes)

| Checkbox | What it enables |
|----------|----------------|
| ✅ **Phase 1** | Generate 57-column CSV box scores |
| ✅ **Phase 2** | Generate `.dat` structured text files |

Untick a phase to skip it entirely.

### 📅 Shared data inputs (apply to all selected phases)

| Input | Description | Default |
|-------|-------------|---------|
| **mode** | How to select games (see below) | `date-range` |
| **start_date** | Start date `YYYY-MM-DD` | `2025-05-13` |
| **end_date** | End date `YYYY-MM-DD` (blank = same as start) | `2025-05-21` |
| **season_year** | Full season year — only used when mode = `season` | `2025` |
| **game_ids** | Space-separated gamePk values — only used when mode = `game-ids` | *(blank)* |

> **No date picker available** in GitHub Actions — type the date directly in `YYYY-MM-DD` format (e.g. `2025-05-13`).

### 🎛️ Data selection modes

| Mode | Uses | Example |
|------|------|---------|
| `date-range` | `start_date` → `end_date` | May 13–21 2025 |
| `single-date` | `start_date` only | One day of games |
| `season` | `season_year` | All 2025 games |
| `game-ids` | `game_ids` (space-separated gamePks) | `777944 777940 777943` |

> **Phase 2 `.dat`** always uses `start_date` → `end_date` regardless of the mode selected.

4. Click **"Run workflow"** — the pipeline runs in ~2 minutes.
5. Download the **artifacts** from the completed workflow run:

| Artifact | Contents |
|----------|----------|
| `phase1-csv-{mode}-{start_date}` | 57-column CSV box score file |
| `phase2-dat-{start_date}` | Structured `.dat` text file(s) |
| `raw-json-{start_date}` | Raw JSON from the MLB Stats API (audit trail) |

---

## Architecture

```
┌──────────────────────┐
│  MLB Stats API       │ <-- Official, free, public API
│  statsapi.mlb.com    │
└──────────┬───────────┘
           │ JSON
           ▼
┌──────────────────────┐
│  Data Scraper        │  Session pooling, retry logic,
│  (Python)            │  rate limiting, caching
└──────────┬───────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
┌────────┐  ┌──────────┐
│  CSV   │  │  .dat    │
│ 57 col │  │ 15 sect  │
└────────┘  └──────────┘
```

Everything runs inside a **pre-built Docker container** pulled from the
GitHub Container Registry — no local setup required.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.12 |
| HTTP Client | `requests` + session pooling |
| Data Transforms | Custom CSV / XML / .dat generators |
| Containerization | Docker (multi-stage build) |
| CI/CD | GitHub Actions |
| Registry | GitHub Container Registry (GHCR) |

---

## FAQ

**Q: Is the source code available?**
A: This repository contains the demo runner only. The full pipeline source code
is packaged inside the Docker container.

**Q: What dates can I use?**
A: Any date during an active MLB season. The API serves historical data back
to ~2017. Opening Day 2025 was March 27.

**Q: Does this cost anything?**
A: No. The MLB Stats API is free and public. GitHub Actions provides free
minutes for public repositories.

**Q: Can I run this locally?**
A: Yes — pull the image and run it:
```bash
docker pull ghcr.io/poornimaramakrishnan/mlb-pipeline:latest

# Date range (Phase 1 CSV)
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --start-date 2025-05-13 --end-date 2025-05-21

# Single date
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --start-date 2025-05-13

# Full season
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --season 2025

# Specific games by gamePk
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --game-id 777944 777940 777943

# .dat structured text
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  dat --date 2025-05-13
```

---

## License

Demo runner — MIT.  Pipeline container image — proprietary.

---

*Built by [Poornima Ramakrishnan](https://github.com/poornimaramakrishnan)*

