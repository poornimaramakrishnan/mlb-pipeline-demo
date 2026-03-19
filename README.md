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
3. Choose a **Phase 1 mode** from the dropdown:

   | Mode | What it does | Key input(s) |
   |------|-------------|-------------|
   | `single-date` | One day of games | `csv_start_date` |
   | `date-range` | Date span of games | `csv_start_date` + `csv_end_date` |
   | `season` | All games in a season | `csv_season` |
   | `game-ids` | Specific games by ID | `csv_game_ids` (space-separated) |

4. Optionally change the **Phase 2 .dat date range** (default: `2022-09-01` to `2022-09-03`).
   Leave `dat_end_date` blank (or equal to `dat_start_date`) to generate a single day.
5. Wait ~60–90 seconds for the workflow to complete.
6. Download the **artifacts** from the workflow run:

| Artifact | Contents |
|----------|----------|
| `phase1-csv-{mode}` | 57-column CSV box score file |
| `phase2-dat-{date}` | Structured `.dat` text file |
| `raw-json-api-responses` | Raw JSON from the MLB Stats API (audit trail) |

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

# Single date
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --start-date 2025-04-01

# Date range
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --start-date 2025-04-01 --end-date 2025-04-07

# Full season
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --season 2025

# Specific games by ID
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --game-id 745456 745457 745458

# .dat structured text
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  dat --date 2022-09-01
```

---

## License

Demo runner — MIT.  Pipeline container image — proprietary.

---

*Built by [Poornima Ramakrishnan](https://github.com/poornimaramakrishnan)*

