# IPL Analytics — Data Pipeline & Power BI Dashboard

A complete end-to-end data pipeline for Indian Premier League (IPL) cricket analytics covering **19 seasons (2008–2026)** with **1,212 matches** and **288,226 ball-by-ball deliveries**.

---

## Project Structure

```
IPL DATA/
│
├── ball_by_ball_data.csv              ← Original raw ball-by-ball data
├── ipl_matches_data.csv               ← Original raw match data
├── players-data-updated.csv           ← Original raw player data
├── team_aliases.csv                   ← Original raw team aliases
├── teams_data.csv                     ← Original raw team data
│
├── .env                               ← MySQL credentials (DO NOT commit to Git)
├── .gitignore                         ← Excludes .env from version control
├── change_log.md                      ← Full audit trail of every data change
├── ipl_eda_full_report.md             ← EDA summary report
│
├── data/
│   ├── cleaned/                       ← Stage 1 output: cleaned CSVs
│   │   ├── ball_by_ball_data_clean.csv
│   │   ├── ipl_matches_data_clean.csv
│   │   ├── players-data-updated_clean.csv
│   │   ├── team_aliases_clean.csv
│   │   └── teams_data_clean.csv
│   │
│   └── powerbi_tables/                ← Stage 2 output: Star Schema for Power BI
│       ├── dim_teams.csv
│       ├── dim_players.csv
│       ├── dim_venues.csv
│       ├── dim_date.csv
│       ├── fact_matches.csv
│       └── fact_deliveries.csv
│
└── notebooks/
    └── eda/
        ├── 01_ball_by_ball_eda.ipynb      ← EDA on raw ball-by-ball data
        ├── 02_ipl_matches_eda.ipynb       ← EDA on raw match data
        ├── 03_players_eda.ipynb           ← EDA on raw player data
        ├── 04_team_aliases_eda.ipynb      ← EDA on raw team aliases
        ├── 05_teams_eda.ipynb             ← EDA on raw team data
        └── 06_sql_database_setup.ipynb    ← Builds Star Schema in MySQL
```

---

## Pipeline Overview

The project runs in two sequential stages:

```
Raw CSVs (root)
      │
      ▼
[STAGE 1] execute_cleaning.py  ──►  data/cleaned/*_clean.csv
      │
      ▼
[STAGE 2] 06_sql_database_setup.ipynb  ──►  data/powerbi_tables/  +  MySQL: ipl_analytics
```

> EDA notebooks 01–05 are **read-only analysis**. They read the raw CSVs and do not feed into the pipeline.

---

## Stage 1 — Data Cleaning (Raw → Cleaned)

**Script:** `execute_cleaning.py` (standalone Python script, run once before anything else)
**Input:** 5 raw CSVs in the project root
**Output:** 5 cleaned CSVs in `data/cleaned/`

### Key Fixes Applied

#### `ipl_matches_data_clean.csv` (1,212 rows, 30 cols)

| Fix | Detail |
|-----|--------|
| Franchise separation | Deccan Chargers (2008–2012) kept separate from Sunrisers Hyderabad (2013+). `team_id 2` remapped to `1068` for seasons up to 2012 |
| Venue canonicalization | 59 raw venue strings collapsed to 36 canonical physical cricket grounds. Raw name preserved in `venue_raw` |
| City null fill | 51 null cities filled: Dubai (33 rows), Sharjah (18 rows) |
| City standardization | Bangalore → Bengaluru; DY Patil matches → Navi Mumbai |
| `is_playoff` added | True for 70 playoff matches where `match_number` is null |
| `is_complete_season` added | False for in-progress 2026 season (43 matches) |
| `season_label` added | Human-readable label e.g. "2026 (partial, 43 matches)" |
| `is_no_result` added | True for 9 weather/abandoned matches |
| `venue_id` added | Integer FK (1–36) for each canonical ground |
| `venue_raw`, `city_raw` added | Original strings preserved for traceability |

#### `ball_by_ball_data_clean.csv` (288,226 rows, 33 cols)

| Fix | Detail |
|-----|--------|
| Franchise separation | `team_batting` and `team_bowling` remapped from `2` to `1068` for Deccan Chargers deliveries |
| `is_legal_ball` added | False for wides/no-balls (11,811 rows), True otherwise |
| `legal_ball_number` added | 1–6 delivery index per over for legal balls |
| `bowler_type_raw` added | Preserves original before casing standardization |
| `bowler_type` standardized | Casing fixed for 73 rows |

#### `players-data-updated_clean.csv` (799 rows, 6 cols)

| Fix | Detail |
|-----|--------|
| Blank → NaN | 612 `field_pos` and 54 `bowl_style` whitespace-only strings converted to NaN |
| Role swap corrections | 3 players had bowl_style stored in `field_pos` — corrected |
| Casing | 1 `bowl_style` value standardized |

#### `team_aliases_clean.csv`
- Deccan Chargers alias remapped from `team_id = 2` to `team_id = 1068`

---

## Stage 2 — Star Schema Setup (Cleaned → Power BI Tables + MySQL)

**Notebook:** `notebooks/eda/06_sql_database_setup.ipynb`
**Input:** 5 cleaned CSVs from `data/cleaned/`
**Output:**
- 6 CSV files in `data/powerbi_tables/` (load directly into Power BI)
- MySQL database `ipl_analytics` with the same Star Schema

### Star Schema

```
                    dim_date ─────────────────────────────┐
                                                           │
         dim_teams ────────────────────────────────────────┤
                                                           │
         dim_venues ───────────────────────────────────────┤──── fact_matches (1,212 rows)
                                                           │
         dim_players ──────────────────────────────────────┤
                                                           │
                                                           └──── fact_deliveries (288,226 rows)
```

### Dimension Tables

| Table | Rows | Key Columns |
|-------|------|-------------|
| `dim_teams` | 16 | `team_id`, `team_name`, `canonical_name`, `short_name`, `is_active` |
| `dim_players` | 799 | `player_id`, `player_name`, `player_full_name`, `bat_style`, `bowl_style`, `field_pos`, `is_wicketkeeper` |
| `dim_venues` | 36 | `venue_id`, `venue_name`, `city` |
| `dim_date` | 920 | `date_id`, `full_date`, `year`, `month`, `quarter`, `season` |

### Fact Tables

| Table | Rows | Grain |
|-------|------|-------|
| `fact_matches` | 1,212 | One row per match |
| `fact_deliveries` | 288,226 | One row per ball bowled |

> **No SQL views are used.** All aggregations (batting averages, strike rates, economy rates, team totals) are computed using **DAX measures in Power BI** directly over the fact tables.

---

## Using the Data in Power BI

### Option A — Direct CSV (no MySQL required)

1. Open Power BI Desktop
2. **Get Data → Text/CSV**
3. Load all 6 files from `data/powerbi_tables/`
4. In **Model view**, create these relationships:

| From | To |
|------|----|
| `fact_matches[venue_id]` | `dim_venues[venue_id]` |
| `fact_matches[team1]` | `dim_teams[team_id]` |
| `fact_matches[team2]` | `dim_teams[team_id]` |
| `fact_matches[toss_winner]` | `dim_teams[team_id]` |
| `fact_matches[match_winner]` | `dim_teams[team_id]` |
| `fact_deliveries[match_id]` | `fact_matches[match_id]` |
| `fact_deliveries[batter]` | `dim_players[player_name]` |
| `fact_deliveries[bowler]` | `dim_players[player_name]` |

### Option B — MySQL Database

1. Create a `.env` file in the project root with:
   ```
   HOST=localhost
   PORT=3306
   USER=root
   PASSWORD=your_password_here
   ```
2. Run all cells in `notebooks/eda/06_sql_database_setup.ipynb`
3. In Power BI: **Get Data → MySQL Database → localhost → ipl_analytics**

---

## EDA Notebooks (01–05) — Reference Only

These notebooks are **standalone analysis tools**. They read the original raw CSVs and are not part of the data pipeline. Running them does not affect any cleaned or Power BI files.

| Notebook | Reads From | Purpose |
|----------|-----------|---------|
| `01_ball_by_ball_eda.ipynb` | `ball_by_ball_data.csv` | Delivery patterns, extras, wicket analysis |
| `02_ipl_matches_eda.ipynb` | `ipl_matches_data.csv` | Season trends, venue analysis, results |
| `03_players_eda.ipynb` | `players-data-updated.csv` | Player roles, batting and bowling styles |
| `04_team_aliases_eda.ipynb` | `team_aliases.csv` | Team name variants and franchise history |
| `05_teams_eda.ipynb` | `teams_data.csv` | Active vs defunct franchise overview |

---

## Data Quality Summary

All 16 CSV files were verified. Every file has **zero duplicate rows** and **zero broken foreign key references**.

| File | Rows | Duplicate Check | Key Used |
|------|------|-----------------|----------|
| `ball_by_ball_data_clean.csv` | 288,226 | PASS | match_id + innings + over + ball + batter + bowler |
| `ipl_matches_data_clean.csv` | 1,212 | PASS | match_id |
| `players-data-updated_clean.csv` | 799 | PASS | player_id |
| `team_aliases_clean.csv` | 46 | PASS | team_id + alias_name |
| `teams_data_clean.csv` | 16 | PASS | team_id |
| `dim_teams.csv` | 16 | PASS | team_id |
| `dim_players.csv` | 799 | PASS | player_id |
| `dim_venues.csv` | 36 | PASS | venue_id |
| `dim_date.csv` | 920 | PASS | date_id |
| `fact_matches.csv` | 1,212 | PASS | match_id |
| `fact_deliveries.csv` | 288,226 | PASS | match_id + innings + over + ball + batter + bowler |

Full change audit: see `change_log.md`

---

## Dataset Coverage

| Metric | Value |
|--------|-------|
| Seasons | 2008–2026 (19 seasons; 2026 is partial — 43 matches) |
| Matches | 1,212 |
| Deliveries | 288,226 |
| Players | 799 |
| Teams (all time) | 16 |
| Venues (canonical physical grounds) | 36 |
#   i p l - d a t a - a n a l y s i s  
 