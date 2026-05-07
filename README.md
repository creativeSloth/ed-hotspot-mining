# ed-hotspot-mining

A laser-mining assistant for [Elite Dangerous](https://www.elitedangerous.com/). Finds nearby ring hotspots with multiple overlapping mineral signals and matches them with the best stations to sell to — based on the player's current location, ship jump range, and landing-pad requirements.

> **Status:** active development. Currently building the data layer (Spansh galaxy dump → SQLite). No runnable tool yet.

## Goals

- Identify the closest ring hotspots within jump range, prioritising rings with **two or more overlapping signals** (double/triple-overlap rings yield significantly higher mining throughput).
- Surface the best-paying, reachable stations for each candidate commodity, taking ship pad size and station distance into account.
- Run fully offline after the initial setup — no per-query API rate limits, sub-second hotspot lookups.
- Stay current with minimal maintenance: the tool detects local-database age and pulls the smallest available delta dump on startup.

## Architecture

A hybrid data strategy reflecting the different update cadences of the data the tool needs:

| Data | Source | Update cadence |
| --- | --- | --- |
| Systems, bodies, rings, hotspot signals, stations, market prices | [Spansh galaxy dump](https://spansh.co.uk/dumps) (`galaxy_populated.json.gz`), parsed into local SQLite | Daily / weekly delta, depending on local DB age |
| Commander position, current ship, jump range | Local Elite Dangerous journal file | Real-time (file watcher) |
| Live market prices (optional, future) | EDSM / Inara REST API | On-demand |

On launch, the tool inspects the local DB age and selects the appropriate Spansh delta (`galaxy_1day`, `galaxy_7days`, `galaxy_1month`) or a full re-import if the database is too stale to patch.

## Tech stack

- Python 3.11+
- [SQLAlchemy 2.0](https://docs.sqlalchemy.org/en/20/) (ORM, SQLite backend)
- [`requests`](https://requests.readthedocs.io/) for REST calls
- [`ijson`](https://pypi.org/project/ijson/) for streaming-parse of the multi-GB dump
- [PyQt6](https://pypi.org/project/PyQt6/) / PySide6 for the desktop UI (later phase)

## Setup

```bash
git clone <repo-url>
cd ed-hotspot-mining
python -m venv .venv
source .venv/bin/activate         # Linux / macOS
# .venv\Scripts\activate          # Windows
pip install -r requirements.txt
```

A first launch will download the Spansh `galaxy_populated.json.gz` dump (~3.6 GB) and build the local SQLite database. Subsequent launches use the existing DB and apply only deltas.

## Requirements

Pinned dependencies live in [`requirements.txt`](requirements.txt) and are updated as the project evolves.

## Roadmap

### Core (planned)
- [x] REST API exploration & first calls (EDSM, Spansh)
- [ ] **Bulk dump → SQLite import (current)**
- [ ] Journal watcher for commander position, ship, jump range
- [ ] Distance / ranking logic for nearby hotspots
- [ ] Station + market-price matching for sale targets
- [ ] CLI interface, dump-refresh manager
- [ ] Desktop GUI (PyQt6)

### Possible feature extensions
- Profit estimation per run (refinery slots, cargo capacity, sell price, station distance, fuel)
- Route planning across multiple hotspots and a sale destination, with refuel stops
- Mining-session tracking: tons mined per ring, average yield, history
- Live EDDN subscriber to keep prices fresh between dump downloads
- Alerts when a tracked hotspot is rescanned (signal changes) or when a configured commodity exceeds a threshold price within range
- Multi-commander support with per-profile filters (preferred minerals, max station distance, pad size, etc.)
- CSV / clipboard export of search results for sharing or in-game pasting
- Dark / light themes, configurable layouts

## Data sources

- [Spansh](https://spansh.co.uk) — aggregated galaxy dataset (bulk dumps + REST API)
- [EDSM](https://www.edsm.net) — system lookup and sphere search
- [EDDN](https://eddn.edcd.io) — real-time event stream from in-game tools (longer-term integration target)

## License

To be determined.
