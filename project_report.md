# IPL Analytics — Full Project Report

**Project:** IPL Data Pipeline & Power BI Star Schema
**Dataset:** 19 seasons (2008–2026) | 1,212 matches | 288,226 deliveries
**Status:** ✅ Complete — Ready for Power BI

---

## What Was Done — Summary

```
Phase 1 → Data Quality Audit & Fixes
Phase 2 → Deduplication & File Cleanup
Phase 3 → Directory Reorganization
Phase 4 → Notebook 06 Alignment
Phase 5 → README & Power BI Guide
```

---

## Phase 1 — Data Quality Fixes (29 Changes)

All changes are non-destructive. Originals are untouched. Cleaned files are in `data/cleaned/`.

### A. `ipl_matches_data_clean.csv` — 15 Fixes

| # | Column | Rows Affected | Before | After |
|---|--------|--------------|--------|-------|
| 1 | `team1` | 39 | team_id = 2 (SRH) for seasons ≤ 2012 | team_id = 1068 (Deccan Chargers) |
| 2 | `team2` | 36 | team_id = 2 (SRH) for seasons ≤ 2012 | team_id = 1068 (Deccan Chargers) |
| 3 | `toss_winner` | 43 | team_id = 2 for seasons ≤ 2012 | team_id = 1068 |
| 4 | `match_winner` | 29 | team_id = 2 for seasons ≤ 2012 | team_id = 1068 |
| 5 | `venue_raw` *(added)* | 1,212 | Column did not exist | Raw venue string copied before normalization |
| 6 | `venue` | 1,212 | 59 raw string variants | Normalized to 36 canonical physical grounds |
| 7 | `venue_id` *(added)* | 1,212 | Column did not exist | Integer FK 1–36 per canonical ground |
| 8 | `city_raw` *(added)* | 1,212 | Column did not exist | Original city string preserved |
| 9 | `city` (null fill) | 33 | null for Dubai International Cricket Stadium | "Dubai" |
| 10 | `city` (null fill) | 18 | null for Sharjah Cricket Stadium | "Sharjah" |
| 11 | `city` (standardize) | 65 | "Bangalore" | "Bengaluru" |
| 12 | `city` (standardize) | 37 | "Mumbai" for Dr DY Patil matches | "Navi Mumbai" |
| 13 | `is_playoff` *(added)* | 1,212 | Column did not exist | True for 70 playoff matches (match_number is null) |
| 14 | `is_complete_season` *(added)* | 1,212 | Column did not exist | False for 2026 season (43 matches), True for 2008–2025 |
| 15 | `season_label` *(added)* | 1,212 | Column did not exist | e.g. "2026 (partial, 43 matches)" |
| 16 | `is_no_result` *(added)* | 1,212 | Column did not exist | True for 9 no-result/abandoned matches |
| 17 | `city` (venue_id 19) | 15 | "New Chandigarh" for Mullanpur stadium | "Mohali" (standardized to remove duplicate venue_id) |

### B. `ball_by_ball_data_clean.csv` — 5 Fixes

| # | Column | Rows Affected | Before | After |
|---|--------|--------------|--------|-------|
| 1 | `team_batting` | 9,034 | team_id = 2 for seasons ≤ 2012 | team_id = 1068 (Deccan Chargers) |
| 2 | `team_bowling` | 9,039 | team_id = 2 for seasons ≤ 2012 | team_id = 1068 |
| 3 | `is_legal_ball` *(added)* | 288,226 | Column did not exist | False for wides/no-balls (11,811 rows), True for 276,415 |
| 4 | `legal_ball_number` *(added)* | 288,226 | Column did not exist | 1–6 delivery index per over for legal balls |
| 5 | `bowler_type_raw` *(added)* | 288,226 | Column did not exist | Raw bowler type preserved before casing fix |
| 6 | `bowler_type` (casing) | 73 | "Right arm Fast Medium" | "Right arm Fast medium" |

### C. `players-data-updated_clean.csv` — 6 Fixes

| # | Column | Rows Affected | Before | After |
|---|--------|--------------|--------|-------|
| 1 | `field_pos` | 612 | Whitespace-only strings | NaN |
| 2 | `bowl_style` | 54 | Whitespace-only strings | NaN |
| 3 | `field_pos` + `bowl_style` (MJ Owen) | 1 | bowl_style in wrong column | Swapped to correct column |
| 4 | `field_pos` + `bowl_style` (R Minz) | 1 | bowl_style in wrong column | Swapped to correct column |
| 5 | `field_pos` (W O'Rourke) | 1 | Redundant value matching bowl_style | Cleared to NaN |
| 6 | `bowl_style` (casing) | 1 | "Right arm Fast Medium" | "Right arm Fast medium" |

### D. `team_aliases_clean.csv` — 1 Fix

| # | Column | Rows Affected | Before | After |
|---|--------|--------------|--------|-------|
| 1 | `team_id` | 1 | team_id = 2 for "Deccan Chargers" alias | team_id = 1068 |

> **Franchise Logic Applied:** Deccan Chargers (2008–2012) and Sunrisers Hyderabad (2013+) are **separate franchises** — not the same team. Delhi Daredevils = Delhi Capitals (rename). Kings XI = Punjab Kings (rename). Rising Pune Supergiant vs Supergiants kept separate.

---

## Phase 2 — Deduplication Audit

Full audit run against logical primary keys. **Every file passed with zero duplicates.**

| File | Rows | Key | Result |
|------|------|-----|--------|
| `ball_by_ball_data_clean.csv` | 288,226 | match_id + innings + over + ball + batter + bowler | ✅ PASS |
| `ipl_matches_data_clean.csv` | 1,212 | match_id | ✅ PASS |
| `players-data-updated_clean.csv` | 799 | player_id | ✅ PASS |
| `team_aliases_clean.csv` | 46 | team_id + alias_name | ✅ PASS |
| `teams_data_clean.csv` | 16 | team_id | ✅ PASS |
| `dim_teams.csv` | 16 | team_id | ✅ PASS |
| `dim_players.csv` | 799 | player_id | ✅ PASS |
| `dim_venues.csv` | 36 | venue_id | ✅ PASS |
| `dim_date.csv` | 920 | date_id | ✅ PASS |
| `fact_matches.csv` | 1,212 | match_id | ✅ PASS |
| `fact_deliveries.csv` | 288,226 | match_id + innings + over + ball + batter + bowler | ✅ PASS |

---

## Phase 3 — Directory Reorganization

Eliminated stale intermediate files. Project now has exactly **16 CSV files** across 3 clearly defined locations.

### Before (21 files, scattered)
```
root/                         ← 10 files (5 original + 5 _clean.csv mixed together)
data/cleaned/                 ← 5 stale files (old pipeline, fewer columns)
data/powerbi_tables/          ← 10 files (4 agg views + 4 dims + 2 facts)
```

### After (16 files, organized)
```
root/                         ← 5 original raw files only
data/cleaned/                 ← 5 updated _clean.csv files (full columns)
data/powerbi_tables/          ← 4 dim tables + 2 fact tables only
```

### Files Deleted
| File | Reason |
|------|--------|
| `data/cleaned/ball_by_ball_cleaned.csv` | Old stale file (30 cols vs 33 in updated clean) |
| `data/cleaned/ipl_matches_cleaned.csv` | Old stale file (15 cols vs 30 in updated clean) |
| `data/cleaned/players_data_cleaned.csv` | Duplicate of updated clean |
| `data/cleaned/team_aliases_cleaned.csv` | Duplicate of updated clean |
| `data/cleaned/teams_data_cleaned.csv` | Duplicate of updated clean |
| `data/powerbi_tables/agg_batting.csv` | Aggregation view — not a fact/dimension table |
| `data/powerbi_tables/agg_bowling.csv` | Aggregation view — not a fact/dimension table |
| `data/powerbi_tables/agg_over.csv` | Aggregation view — not a fact/dimension table |
| `data/powerbi_tables/agg_team_match.csv` | Aggregation view — not a fact/dimension table |

### Files Rebuilt
| File | Before | After |
|------|--------|-------|
| `data/powerbi_tables/fact_matches.csv` | Stale SQL-derived (17 cols, missing venue/city/playoff columns) | Rebuilt from `ipl_matches_data_clean.csv` (30 cols) |
| `data/powerbi_tables/fact_deliveries.csv` | Stale SQL-derived (38 cols, missing legal_ball/team columns) | Rebuilt from `ball_by_ball_data_clean.csv` (33 cols) |
| `data/powerbi_tables/dim_venues.csv` | 59 rows with raw venue names + 1 duplicate venue_id | 36 canonical physical grounds, 0 duplicates, 0 null cities |
| `data/powerbi_tables/dim_teams.csv` | `short_name` was NaN for Deccan Chargers | Populated with "DC" |

---

## Phase 4 — Notebook Alignment

### EDA Notebooks 01–05 — No Changes
These read the original raw CSVs for analysis only. No changes needed or made.

### `06_sql_database_setup.ipynb` — Fully Rewritten

| # | Change | Before | After |
|---|--------|--------|-------|
| 1 | **Password security** | `PASSWORD = 'Alladin@123'` hardcoded in plain text (Cell 4) | Loaded from `.env` via `python-dotenv`. `.env` added to `.gitignore` |
| 2 | **Data source** | Read from `data/cleaned/ball_by_ball_cleaned.csv` (stale, 15 cols) | Read from `data/cleaned/ball_by_ball_data_clean.csv` (current, 33 cols) |
| 3 | **Views removed** | Created 4 SQL aggregation views | No views created — Star Schema tables only |
| 4 | **`fact_matches` schema** | 17 columns | 19 columns (adds `is_playoff`, `is_complete_season`) |
| 5 | **`fact_deliveries` schema** | Missing `is_legal_ball`, `legal_ball_number` | Includes all clean columns |
| 6 | **Sample queries** | Queried through `agg_batting`, `agg_bowling` views | Query directly against `fact_deliveries` + `dim_*` tables |
| 7 | **Cell count** | 49 cells | 39 cells (cleaner, no view steps) |

---

## Phase 5 — Documentation Created

| File | What it contains |
|------|-----------------|
| [`README.md`](file:///c:/Users/Bhuvan/Desktop/IPL%20DATA/README.md) | Full project guide: structure, pipeline, how to run, Power BI setup, DAX measures |
| [`change_log.md`](file:///c:/Users/Bhuvan/Desktop/IPL%20DATA/change_log.md) | Audit trail of all 29 data fixes across 3 phases |
| [`.gitignore`](file:///c:/Users/Bhuvan/Desktop/IPL%20DATA/.gitignore) | Excludes `.env` and Python cache from version control |

---

## Final Project State

### File Inventory

```
c:\Users\Bhuvan\Desktop\IPL DATA\
│
├── ball_by_ball_data.csv              288,226 rows  42.9 MB  [RAW]
├── ipl_matches_data.csv                 1,212 rows   0.2 MB  [RAW]
├── players-data-updated.csv               799 rows   0.1 MB  [RAW]
├── team_aliases.csv                        46 rows   0.0 MB  [RAW]
├── teams_data.csv                          16 rows   0.0 MB  [RAW]
│
├── .env                                                       [CREDENTIALS]
├── .gitignore                                                 [GIT]
├── README.md                                                  [DOCUMENTATION]
├── change_log.md                                              [AUDIT TRAIL]
├── ipl_eda_full_report.md                                     [EDA REPORT]
│
├── data/cleaned/
│   ├── ball_by_ball_data_clean.csv    288,226 rows  49.7 MB  [CLEANED]
│   ├── ipl_matches_data_clean.csv       1,212 rows   0.2 MB  [CLEANED]
│   ├── players-data-updated_clean.csv     799 rows   0.1 MB  [CLEANED]
│   ├── team_aliases_clean.csv              46 rows   0.0 MB  [CLEANED]
│   └── teams_data_clean.csv                16 rows   0.0 MB  [CLEANED]
│
├── data/powerbi_tables/
│   ├── dim_teams.csv                       16 rows   0.0 MB  [DIMENSION]
│   ├── dim_players.csv                    799 rows   0.1 MB  [DIMENSION]
│   ├── dim_venues.csv                      36 rows   0.0 MB  [DIMENSION]
│   ├── dim_date.csv                       920 rows   0.0 MB  [DIMENSION]
│   ├── fact_matches.csv                 1,212 rows   0.2 MB  [FACT]
│   └── fact_deliveries.csv            288,226 rows  49.7 MB  [FACT]
│
└── notebooks/eda/
    ├── 01_ball_by_ball_eda.ipynb                              [EDA — read-only]
    ├── 02_ipl_matches_eda.ipynb                               [EDA — read-only]
    ├── 03_players_eda.ipynb                                   [EDA — read-only]
    ├── 04_team_aliases_eda.ipynb                              [EDA — read-only]
    ├── 05_teams_eda.ipynb                                     [EDA — read-only]
    └── 06_sql_database_setup.ipynb                            [PIPELINE — updated]
```

### Star Schema

```
             dim_date
                │ date_id
                │
    dim_teams ──┤ team_id                ┌── fact_matches (1,212 rows)
                │                        │   match_id, venue_id, team1, team2,
    dim_venues ─┤ venue_id ──────────────┤   toss_winner, match_winner,
                │                        │   is_playoff, is_complete_season
    dim_players ┘ player_name            │
                                         └── fact_deliveries (288,226 rows)
                                             match_id, batter, bowler,
                                             batter_runs, is_wicket,
                                             is_legal_ball, match_phase
```

### Referential Integrity (Star Schema) — All Clean

| Check | Result |
|-------|--------|
| Deliveries without matching match | ✅ 0 |
| Matches without valid venue_id | ✅ 0 |
| Matches without valid date | ✅ 0 |
| Matches with invalid team1 | ✅ 0 |
| Matches with invalid team2 | ✅ 0 |

---

## What's Next

1. **Open Power BI** → Load 6 files from `data/powerbi_tables/`
2. **Set up relationships** in Model view (see README or previous Power BI guide)
3. **Create DAX measures** for runs, wickets, strike rate, economy rate
4. **Build report pages**: Overview, Batting, Bowling, Teams, Venues, Season view
5. *(Optional)* Run `06_sql_database_setup.ipynb` to also load the Star Schema into MySQL for SQL-based analysis
