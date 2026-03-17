# ⚾ MLB Data Pipeline — Live Demo

> **Real-time MLB statistics pipeline** — scrapes the official MLB Stats API,
> transforms data into production-ready formats, and loads it into a relational
> database.  Click the button below to run it yourself.

[![Run Demo](https://img.shields.io/badge/▶_Run_Demo-Click_Here-blue?style=for-the-badge&logo=github-actions)](../../actions/workflows/demo.yml)
[![Docker Image](https://img.shields.io/badge/Docker-GHCR-2496ED?style=flat-square&logo=docker)](https://ghcr.io/poornimaramakrishnan/mlb-pipeline)

---

## What This Pipeline Does

| Phase | Description | Output |
|-------|-------------|--------|
| **Phase 1** | Scrapes MLB Stats API → 57-column CSV box scores | `.csv` files |
| **Phase 2** | Converts game data → structured `.dat` text (Retrosheet-style) | `.dat` files |
| **Phase 3** | Ingests CSV into SQLite (or MariaDB/PostgreSQL) with reporting | `.db` file |

### Data Columns (Phase 1 CSV — 57 columns)

Game metadata, team info, scores by inning, pitching stats (W/L/S, ERA, K, BB),
batting stats (H, HR, RBI, AVG), fielding stats, and more.

### Structured Text (Phase 2 .dat — 15 sections)

Game header, venue/weather, line scores, pitching lines, batting lines,
fielding summaries, play-by-play highlights, and game notes — all in a
human-readable fixed-width format suitable for legacy system ingestion.

### Database (Phase 3 — 6 tables)

`teams`, `players`, `games`, `standings`, `player_stats`, `game_box_scores`
— fully normalized with foreign keys, indexes, and built-in summary reports.

---

## ▶️  How to Run the Demo

1. Click the **[Run Demo](../../actions/workflows/demo.yml)** badge above
   (or go to the **Actions** tab → **"▶️ Run MLB Pipeline Demo"**).
2. Click the green **"Run workflow"** button.
3. Optionally change the dates:
   - **Phase 1 CSV date** — any date in the current/recent season
   - **Phase 2 .dat date** — any date with MLB games (try `2022-09-01`)
4. Wait ~60–90 seconds for the workflow to complete.
5. Download the **artifacts** from the workflow run:

| Artifact | Contents |
|----------|----------|
| `phase1-csv-boxscores` | 57-column CSV box score file |
| `phase2-dat-structured-text` | Structured `.dat` text file |
| `phase3-sqlite-database` | SQLite database with all ingested data |
| `raw-json-api-responses` | Raw JSON from the MLB Stats API (audit trail) |

---

## Architecture

```
┌──────────────────────┐
│  MLB Stats API       │ ◄── Official, free, public API
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
└───┬────┘  └──────────┘
    │
    ▼
┌──────────────────────┐
│  SQL Database        │  SQLAlchemy ORM
│  (SQLite / MariaDB)  │  6 normalized tables
└──────────────────────┘
```

Everything runs inside a **pre-built Docker container** pulled from the
GitHub Container Registry — no local setup required.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.11 |
| HTTP Client | `requests` + session pooling |
| Data Transforms | Custom CSV / XML / .dat generators |
| Database ORM | SQLAlchemy 2.x |
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
docker run --rm -v "$(pwd)/output:/app/output" \
  ghcr.io/poornimaramakrishnan/mlb-pipeline:latest \
  csv --season 2025 --start-date 2025-04-01 --end-date 2025-04-01
```

---

## License

Demo runner — MIT.  Pipeline container image — proprietary.

---

*Built by [Poornima Ramakrishnan](https://github.com/poornimaramakrishnan)*
