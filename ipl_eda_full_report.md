# IPL Data — Full EDA Report
### All CSV Files · Analysis Steps, Findings & Cleaning Decisions

---

## Overview

| # | CSV File | Rows | Columns | Notebook |
|---|---|---|---|---|
| 1 | `ball_by_ball_data.csv` | 288,226 | 30 | `01_ball_by_ball_eda.ipynb` |
| 2 | `ipl_matches_data.csv` | 1,212 | 23 | `02_ipl_matches_eda.ipynb` |
| 3 | `players-data-updated.csv` | 799 | 6 | `03_players_eda.ipynb` |
| 4 | `team_aliases.csv` | 46 | 3 | `04_team_aliases_eda.ipynb` |
| 5 | `teams_data.csv` | 16 | 2 | `05_teams_eda.ipynb` |

Each dataset was investigated using the same structured methodology:  
**Inspect → Question → Investigate → Decide → Clean → Verify**

---

## Methodology Applied to Every Dataset

### Phase 1 — Structural Inspection
Before any analysis, the following baseline checks were run on every CSV:

1. **Shape** — row count and column count
2. **Column names** — detect naming inconsistencies or unusual casing
3. **Data types** — confirm that pandas inferred types are appropriate
4. **Sample records** — first 5 rows and last 3 rows to understand the grain
5. **Memory usage** — total DataFrame size in MB
6. **Exact duplicate rows** — full-row deduplication check
7. **Missing value count and percentage** — per column
8. **Numeric summary statistics** — `describe()` for all numeric columns
9. **Distinct value counts** — for categorical columns

### Phase 2 — Column-by-Column Investigation
Every column was classified by its role and investigated accordingly:

| Column Role | Checks Applied |
|---|---|
| **Primary key / identifier** | Uniqueness, range, gaps, non-sequential IDs |
| **Numeric measurement** | Min/max range, negatives, impossible values, distribution |
| **Categorical** | Value counts, whitespace padding, blank strings, unexpected categories |
| **Boolean flag** | Distribution of True/False, cross-column consistency |
| **Date / time** | Parse success, date range, format inconsistencies |
| **Free text / name** | Whitespace, empty strings, case inconsistency, duplicates |
| **Foreign key** | Values within expected domain, referential plausibility |

### Phase 3 — Missing-Value Analysis
For each column with nulls or blank strings:
- Count and percentage of affected rows
- Inspect representative affected records
- Determine whether missingness is structurally valid, ambiguous, or a genuine error
- Select treatment: retain, standardise, convert, impute, or flag

### Phase 4 — Domain Validation
Cricket-specific rules applied where relevant:
- Run arithmetic consistency
- Extras breakdown validation
- Wicket flag / dismissal column alignment
- Valid delivery coordinates (over, ball number)
- Toss winner must be one of the two playing teams
- Match winner must be one of the two playing teams
- Wide and no-ball cannot occur on the same delivery

### Phase 5 — Cleaning
Only cleaning supported by evidence from the investigation was applied. No records were deleted without justification. No synthetic values were inserted.

### Phase 6 — Verification
After every cleaning step, immediate verification was performed:
- Row count unchanged (or documented if changed)
- Null counts rechecked
- Cleaned column values inspected
- Key domain rules re-validated

---

---

# Dataset 1 — `ball_by_ball_data.csv`

## Dataset Description

Each row is one ball bowled in an IPL match. The dataset covers 19 IPL seasons (2008–2026)
across 30 columns. It is the most granular dataset in the project.

**File size:** 41.9 MB  
**Rows:** 288,226  
**Columns:** 30  

## Columns

| Column | Type | Role |
|---|---|---|
| `season_id` | int64 | Season year (2008–2026) |
| `match_id` | int64 | Foreign key to matches table |
| `batter` | string | Batter's name |
| `bowler` | string | Bowler's name |
| `non_striker` | string | Non-striking batter's name |
| `team_batting` | int64 | Batting team FK |
| `team_bowling` | int64 | Bowling team FK |
| `over_number` | int64 | Over index (0-based, 0–19 normal; >19 super over) |
| `ball_number` | int64 | Ball index within over (0-based; >5 on wides/no-balls) |
| `batter_runs` | int64 | Runs scored by batter off this delivery |
| `extras` | int64 | Total extras on this delivery |
| `total_runs` | int64 | Total runs off this delivery |
| `batsman_type` | string | Batting hand style |
| `bowler_type` | string | Bowling style |
| `player_out` | string | Dismissed batter's name (null if no wicket) |
| `fielders_involved` | string | Fielder(s) in dismissal (null if no wicket) |
| `is_wicket` | bool | Wicket fell on this delivery |
| `is_wide_ball` | bool | Delivery was a wide |
| `is_no_ball` | bool | Delivery was a no-ball |
| `is_leg_bye` | bool | Leg-bye scored |
| `is_bye` | bool | Bye scored |
| `is_penalty` | bool | Penalty runs awarded |
| `wide_ball_runs` | int64 | Runs from wide |
| `no_ball_runs` | int64 | Runs from no-ball |
| `leg_bye_runs` | int64 | Runs from leg-bye |
| `bye_runs` | int64 | Runs from bye |
| `penalty_runs` | int64 | Penalty runs |
| `wicket_kind` | string | Dismissal type (null if no wicket) |
| `is_super_over` | bool | Delivery is part of a super over |
| `innings` | int64 | Innings number (1–2 normal; 3–6 super overs) |

## Step-by-Step Analysis

### Step 1 — Structure and Shape
- Confirmed 288,226 rows × 30 columns
- Memory usage: ~500 MB (large file — 30 columns with string fields)
- 0 exact duplicate rows
- 19 seasons covered: 2008 through 2026

### Step 2 — Missing Values
| Column | Nulls | % Missing | Why |
|---|---|---|---|
| `batsman_type` | 239 | 0.08% | Player style not available for a small subset |
| `bowler_type` | 339 | 0.12% | Same — player not found in reference data |
| `player_out` | 273,886 | 95.0% | Null = no wicket fell — structurally valid |
| `fielders_involved` | 273,886 | 95.0% | Same |
| `wicket_kind` | 273,886 | 95.0% | Same |

**Key finding:** `player_out`, `wicket_kind`, and `fielders_involved` all have exactly 273,886 nulls — equal to the number of non-wicket deliveries (is_wicket=False). This is confirmed correct.

### Step 3 — Identifier Validation (season_id, match_id)
- Seasons 2008–2026 all represented
- Deliveries per season range from ~9,000 (shorter seasons) to ~20,000+ (longer ones)
- `match_id` is numeric but acts as an FK to the matches table — not necessarily sequential

### Step 4 — Player Columns (batter, bowler, non_striker)
- Checked for whitespace padding using `df[col] != df[col].str.strip()`
- Checked for empty strings using `df[col].str.strip() == ''`
- Counted unique players in each role
- **Finding:** No padding or empty strings found

### Step 5 — Team Identifier Validation (team_batting, team_bowling)
- Listed unique team IDs in both columns
- Verified that `team_batting != team_bowling` on every delivery
- **Finding:** No row has the same team batting and bowling — confirmed valid

### Step 6 — Delivery Coordinate Validation (over_number, ball_number)
- `over_number` range: 0 to 19 (normal play); checked that values >19 only appear in super overs
- `ball_number` range: 0 to 10
- **Investigated:** ball_number > 5 — cross-checked with `is_wide_ball` and `is_no_ball`  
  → All high ball numbers occur only on wides or no-balls, which do not count as legal deliveries in cricket. **Valid.**
- Super overs confirmed: 175 deliveries with `is_super_over=True` and `over_number=20`

### Step 7 — Run Arithmetic Validation (batter_runs, extras, total_runs)
- **Rule tested:** `total_runs == batter_runs + extras`  
  → **0 mismatches found** — arithmetic is fully consistent
- **Rule tested:** `wide_ball_runs + no_ball_runs + leg_bye_runs + bye_runs + penalty_runs == extras`  
  → **0 mismatches** — extras breakdown is fully consistent
- `batter_runs` range: 0–6 (valid; 6 is a six)
- `extras` range: 0–7 (investigated high values — penalty runs account for unusual totals)
- `total_runs` range: 0–7
- **No negative values** found in any run column

### Step 8 — Extra-Type Flag Validation
- Confirmed `is_wide_ball=True` rows always have `wide_ball_runs > 0` and vice versa
- **Found:** 0 deliveries flagged as both wide AND no-ball (correct — illegal in cricket)
- Distributions:
  - is_wide_ball True: ~20,000+ deliveries
  - is_no_ball True: ~7,000+ deliveries
  - is_leg_bye / is_bye / is_penalty: smaller counts, all consistent with run columns

### Step 9 — Wicket Column Validation
- Total wickets: 14,340
- Wicket kind distribution:

| Dismissal Type | Count |
|---|---|
| caught | 9,056 |
| bowled | 2,407 |
| run out | 1,176 |
| lbw | 873 |
| caught and bowled | 399 |
| stumped | 382 |
| retired hurt | 19 |
| hit wicket | 18 |
| retired out | 6 |
| obstructing the field | 4 |

- Wickets on wide deliveries: a small number found — investigated  
  → These are run-outs on wide deliveries, which is legal in cricket. **Retained.**
- Fielders on non-wicket deliveries: 0 rows

### Step 10 — Style Columns (batsman_type, bowler_type)
- `batsman_type`: only 2 values — "Right hand Bat" and "Left hand Bat" (clean)
- `bowler_type`: ~20 distinct bowling styles — all valid cricket categories
- Missing style values investigated: small number of players whose style was not in the player reference

### Step 11 — Innings and Super Over
- Innings 1 & 2: 288,051 deliveries (normal play)
- Innings 3–6: 175 deliveries (all confirmed `is_super_over=True`)

## Cleaning Decisions

| Action | Justification |
|---|---|
| Whitespace stripped from `batter`, `bowler`, `non_striker`, `player_out` | Preventive — no padding found but standardised |
| Whitespace stripped from `batsman_type`, `bowler_type`, `wicket_kind` | Same |
| `player_out` / `wicket_kind` / `fielders_involved` nulls retained | Structurally valid — null = no wicket |
| `batsman_type` / `bowler_type` nulls retained | Cannot impute; requires join with player table |

**Rows removed:** 0  
**Columns removed:** 0  
**Records modified:** 0 (whitespace stripping found nothing to change)

## Unresolved Issues

- `batsman_type` and `bowler_type` gaps remain — resolvable by joining with `players-data-updated.csv` in SQL
- `fielders_involved` contains compound strings (multiple fielders separated by delimiter) — parsing left for SQL layer

---

---

# Dataset 2 — `ipl_matches_data.csv`

## Dataset Description

Each row is one IPL match. Covers 1,212 matches across 19 seasons (2008–2026).
Teams, players, and venues are stored as numeric IDs or strings.

**File size:** 178 KB  
**Rows:** 1,212  
**Columns:** 23  

## Columns

| Column | Type | Role |
|---|---|---|
| `match_id` | int64 | Primary key |
| `season_id` | int64 | Season year |
| `balls_per_over` | int64 | Always 6 |
| `city` | string | Host city (51 nulls) |
| `match_date` | string | Date string (YYYY-MM-DD format) |
| `event_name` | string | Always "Indian Premier League" |
| `match_number` | float64 | Sequential match number within season (70 nulls) |
| `gender` | string | Always "male" |
| `match_type` | string | Always "T20" |
| `format` | string | Always "T20" |
| `overs` | int64 | Always 20 |
| `season` | string | Season year as string |
| `team_type` | string | Always "club" |
| `venue` | string | Full venue name |
| `toss_winner` | int64 | Team ID of toss winner |
| `team1` | int64 | Home/first listed team FK |
| `team2` | int64 | Away/second listed team FK |
| `toss_decision` | string | "bat" or "field" |
| `match_winner` | int64 | Team ID of match winner |
| `win_by_runs` | float64 | Margin in runs (null if win by wickets or non-win) |
| `win_by_wickets` | float64 | Margin in wickets (null if win by runs or non-win) |
| `result` | string | "win", "tie", or "no result" |
| `player_of_match` | float64 | Player ID of awardee (9 nulls) |

## Step-by-Step Analysis

### Step 1 — Structure and Shape
- 1,212 rows × 23 columns
- 0 exact duplicate rows
- `match_id` confirmed fully unique — valid primary key

### Step 2 — Missing Values
| Column | Nulls | % | Reason |
|---|---|---|---|
| `city` | 51 | 4.2% | Venue name present; city extraction not done |
| `match_number` | 70 | 5.8% | Finals/qualifiers in some seasons may not have sequential numbers |
| `win_by_runs` | 666 | 54.9% | Null on wicket-wins and non-wins — structurally correct |
| `win_by_wickets` | 571 | 47.1% | Null on run-wins and non-wins — structurally correct |
| `player_of_match` | 9 | 0.7% | Tied or no-result matches — no award given |

### Step 3 — match_date Parsing
- All dates in YYYY-MM-DD format
- Parsed with `pd.to_datetime()` — 0 parse failures
- Date range: 2008-04-18 to 2026-05-01 (covers full IPL history)

### Step 4 — Constant Columns Detection
**Identified 6 columns with only one unique value:**

| Column | Value |
|---|---|
| `gender` | "male" |
| `match_type` | "T20" |
| `format` | "T20" |
| `team_type` | "club" |
| `balls_per_over` | 6 |
| `overs` | 20 |

These carry no analytical variance. They are retained (not deleted) but noted as candidates for exclusion during SQL modelling.

### Step 5 — city Analysis
- 51 nulls (4.2%)
- `venue` is always present — city can be derived from venue via a lookup table in SQL
- Investigated which venues correspond to null cities — all are recognisable named grounds
- **Decision:** Retain nulls; do not impute without a verified city-to-venue mapping

### Step 6 — venue Analysis
- Unique venues: 59 distinct venue strings across 63 city-venue pairs in raw data, resolving to 36 distinct physical grounds (the previous claim of 34 grounds was inaccurate due to unmerged renamed grounds and suffix variants)
- No whitespace padding found
- Top venues: M Chinnaswamy Stadium, Eden Gardens, Wankhede Stadium, etc. — all valid

### Step 7 — match_number Analysis
- Range: 1 to 72
- 70 nulls investigated by season:
  - These represent playoff matches (qualifiers, eliminators, finals) across ALL completed seasons (3 in 2008 and 2009; 4 in each season from 2010 to 2025). The previous claim that nulls were concentrated in 'early seasons' was incorrect; playoffs have unnumbered match_number in every season.
- **Decision:** Retain nulls; covered by is_playoff flag

### Step 8 — Team and Toss Validation
- `team1` has 14 unique team IDs (all known IPL franchises)
- `toss_winner` checked against `team1` and `team2` on every row:  
  → **0 rows** where toss_winner is not one of the two playing teams ✅
- `match_winner` checked for win matches:  
  → **0 rows** where match_winner is not team1 or team2 ✅

### Step 9 — toss_decision Distribution
| Decision | Count |
|---|---|
| field (bowl first) | 800 |
| bat | 412 |

Field-first is substantially more common — consistent with T20 norms where chasing is generally favoured.

### Step 10 — result and Win Margins
- Result distribution:
  - `win`: 1,187 matches (97.9%)
  - `tie`: 16 matches (1.3%)
  - `no result`: 9 matches (0.7%)

- **Win margin cross-validation:**
  - Win matches with BOTH margins populated: **0** ✅ (mutually exclusive confirmed)
  - Win matches with NEITHER margin populated: **0** ✅
  - Tie/no-result rows with any margin: **0** ✅

- Win by runs range: 1 to 140 runs (140-run win is large but plausible in T20)
- Win by wickets range: 1 to 10 wickets (10 = team 2 chased without losing a wicket)

### Step 11 — player_of_match
- 9 nulls — all occur on tied or no-result matches where no award is given
- Stored as float64 (player ID) — compatible with joining to players table

## Cleaning Decisions

| Action | Justification |
|---|---|
| `match_date` parsed to datetime | Enables date filtering and range queries |
| Whitespace stripped from all string columns | Preventive — no padding found |
| All nulls retained | Each null column has a structural or contextual justification |
| 6 constant columns retained | Not removed — left for SQL layer decision |

**Rows removed:** 0  
**Columns removed:** 0  

## Unresolved Issues

- `city` nulls (51 rows) — resolvable via a venue-to-city lookup
- `match_number` nulls (70 rows) — likely playoff matches; requires schedule metadata to confirm
- Constant columns (`gender`, `format`, etc.) — remove at SQL/dimensional model stage, not here

---

---

# Dataset 3 — `players-data-updated.csv`

## Dataset Description

Player reference table — one row per IPL player. Contains batting style, bowling style,
fielding position, and two name representations per player.

**File size:** 56 KB  
**Rows:** 799  
**Columns:** 6  

## Columns

| Column | Type | Role |
|---|---|---|
| `player_id` | int64 | Primary key |
| `player_name` | string | Short initials-style name (e.g. "DA Warner") |
| `bat_style` | string | "Right hand Bat" or "Left hand Bat" |
| `bowl_style` | string | Bowling style category (19 nulls + blank strings) |
| `field_pos` | string | Fielding position (mostly blank) |
| `player_full_name` | string | Full proper name (e.g. "David Andrew Warner") |

## Step-by-Step Analysis

### Step 1 — Structure and Shape
- 799 rows × 6 columns
- 0 exact duplicate rows
- `player_id` is unique — valid primary key

### Step 2 — Missing Values (Before Cleaning)
| Column | Nulls | Blank Strings | Total Gap |
|---|---|---|---|
| `bowl_style` | 19 | 54 | 73 |
| `field_pos` | 88 | 612 | 700 |

> [!IMPORTANT]
> The blank strings were discovered during investigation — they did not show up in the initial `.isnull()` check. This is why column-by-column investigation is critical.

### Step 3 — player_id Validation
- Range: varies (non-sequential — externally assigned IDs)
- Fully unique: confirmed
- No gaps investigation needed — IDs are not generated sequentially

### Step 4 — player_name and player_full_name
- Checked for whitespace padding → none found
- Checked for blank strings → none found
- **Duplicate player_names investigated:**
  - Some initials-style names repeat (e.g. "RG Sharma") — different players with identical initials
  - `player_full_name` distinguishes them — both rows are valid
- `player_full_name`: no duplicates found

### Step 5 — bat_style
- Only 2 distinct values:
  - "Right hand Bat": 582 players (72.8%)
  - "Left hand Bat": 217 players (27.2%)
- No nulls, no blank strings, no whitespace padding
- **Clean — no action needed**

### Step 6 — bowl_style
- 20+ distinct bowling style categories
- 19 genuine NaN values — players with no bowling style recorded
- **54 blank string values discovered** — players where `bowl_style = ""`
- Composite styles found: e.g. `"Right arm Medium, Right arm Offbreak"` — legitimate multi-style entries, not errors
- **Decision:** Convert blank strings to NaN for consistency

Top bowling styles:
| Style | Count |
|---|---|
| Right arm Medium | 141 |
| Right arm Offbreak | 129 |
| Right arm Fast medium | 83 |
| Slow Left arm Orthodox | 68 |
| Right arm Medium fast | 60 |
| (blank string) | 54 |
| Right arm Fast | 52 |
| Legbreak | 48 |

### Step 7 — field_pos
- Only 4 non-blank categories:
  - "Wicketkeeper": 82 players
  - "Occasional Wicketkeeper": 13 players
  - "Wicketkeeper Batter": 1 player
- 88 genuine NaN values
- **612 blank strings** — most players have no fielding position recorded
- **Critical finding:** 3 rows where `field_pos` contains a bowling style string:
  - E.g. `field_pos = "Right arm Medium"` — clearly a data-entry error
  - These players have a valid `bowl_style` in the correct column
  - **Decision:** Set `field_pos` to NaN for these 3 rows

### Step 8 — Style Distribution Chart
- Produced a bar chart of batting style distribution (Right/Left hand)
- Confirmed visual consistency with value counts

## Cleaning Decisions

| Action | Rows Affected | Justification |
|---|---|---|
| Whitespace stripped from all string columns | 0 (none found) | Preventive |
| Blank strings → NaN in `bowl_style` | 54 | Consistency; blank ≠ "no bowling style" in pandas |
| Blank strings → NaN in `field_pos` | 612 | Same reason |
| 3 rows with bowling text in `field_pos` set to NaN | 3 | Confirmed data-entry error |

**Rows removed:** 0  
**Records modified:** 3 (field_pos correction)  

## Unresolved Issues

- Non-bowlers (batters, wicketkeepers) legitimately have null `bowl_style` — cannot impute
- `field_pos` blank for 700 players — this is the actual state of the data; not imputable without external source
- `player_name` collisions (identical initials) — disambiguation possible via `player_id` or `player_full_name`

---

---

# Dataset 4 — `team_aliases.csv`

## Dataset Description

Lookup table mapping every known IPL team name variant to a canonical `team_id`.
Covers abbreviations, city names, historical names, and franchise rebrands.

**File size:** 1 KB  
**Rows:** 46  
**Columns:** 3  

## Columns

| Column | Type | Role |
|---|---|---|
| `alias_id` | int64 | Surrogate primary key (sequential 1–46) |
| `team_id` | int64 | Foreign key to teams_data |
| `alias_name` | string | Name variant (abbreviation, city, historical name) |

## Step-by-Step Analysis

### Step 1 — Full Dataset Review
The table is small enough (46 rows) to inspect in its entirety. Every row was read and classified.

### Step 2 — Missing Values and Duplicates
- Null values: **0** across all columns
- Exact duplicate rows: **0**
- `alias_id` confirmed unique and sequential (1 to 46)

### Step 3 — alias_id
- Sequential: 1 to 46 with no gaps — confirmed

### Step 4 — team_id Coverage
- Unique team IDs referenced: 14
- Aliases per team:

| team_id | Team | Alias Count |
|---|---|---|
| 1 | Royal Challengers Bangalore | 6 |
| 2 | Sunrisers Hyderabad | 4 |
| 3 | Mumbai Indians | 3 |
| 4 | Rising Pune Supergiant | 4 |
| 5 | Gujarat Lions | 2 |
| 6 | Kolkata Knight Riders | 3 |
| 129 | Chennai Super Kings | 3 |
| 134 | Rajasthan Royals | 3 |
| 252 | Delhi Capitals | 5 |
| 494 | Punjab Kings | 4 |
| 614 | Lucknow Super Giants | 3 |
| 615 | Gujarat Titans | 2 |
| 1414 | Kochi Tuskers Kerala | 3 |
| 1419 | Pune Warriors | 2 |

### Step 5 — alias_name Analysis
- Checked for whitespace padding → none found
- Checked for blank strings → none found
- Duplicate alias_name check:
  - Some abbreviations like "RCB", "MI", "CSK" are unique
  - City-only names like "Delhi", "Chennai", "Mumbai" appear once each
  - No alias_name is shared between two different team_ids

### Step 6 — Manual Review of Aliases (Every Row)
A full alias-by-team review surfaced these findings:

**Royal Challengers Bangalore (team_id=1):**
- RCB, Royal Challengers Bengaluru, Royal Challengers Bangalore, Banglore, Bengaluru
- Note: "Banglore" is a misspelling — retained as a historical alias used in source data

**Delhi Capitals (team_id=252):**
- Delhi Daredevils (historical name), Delhi Capitals (current name), DC, DD, Delhi
- Correct — the franchise was renamed from Daredevils to Capitals in 2019

**Punjab Kings (team_id=494):**
- Kings XI Punjab (historical), Punjab Kings (current), KXIP, Punjab
- Correct — renamed from KXIP to Punjab Kings in 2021

**Sunrisers Hyderabad (team_id=2):**
- SRH, Hyderabad, Sunrisers Hyderabad, **Deccan Chargers**
- ⚠️ Notable: Deccan Chargers is mapped here as an alias for SRH (team_id=2)
  But `teams_data.csv` contains a separate entry for Deccan Chargers at team_id=1068
  This is a modelling inconsistency — documented but not altered here

### Step 7 — Whitespace Check
- All 46 alias_name values passed the strip check — no changes required

## Cleaning Decisions

| Action | Result |
|---|---|
| Whitespace check on `alias_name` | No padding found — no changes |

**No cleaning was applied.** The dataset is clean.

## Unresolved Issues

- **Deccan Chargers franchise conflict:** Mapped under SRH (team_id=2) in aliases, but has a separate team_id=1068 in teams_data. This inconsistency must be resolved in the SQL modelling layer.
- team_id=1068 (Deccan Chargers in teams_data) is not referenced in team_aliases at all.

---

---

# Dataset 5 — `teams_data.csv`

## Dataset Description

Master reference table for IPL franchises. Maps numeric `team_id` to canonical team name.
Covers all 16 franchises including defunct ones (Kochi, Deccan Chargers, Rising Pune, etc.).

**File size:** 425 bytes  
**Rows:** 16  
**Columns:** 2  

## Columns

| Column | Type | Role |
|---|---|---|
| `team_id` | int64 | Primary key |
| `team_name` | string | Canonical franchise name |

## Full Dataset

| team_id | team_name |
|---|---|
| 1 | Royal Challengers Bangalore |
| 2 | Sunrisers Hyderabad |
| 3 | Mumbai Indians |
| 4 | Rising Pune Supergiant |
| 5 | Gujarat Lions |
| 6 | Kolkata Knight Riders |
| 129 | Chennai Super Kings |
| 134 | Rajasthan Royals |
| 252 | Delhi Capitals |
| 494 | Punjab Kings |
| 614 | Lucknow Super Giants |
| 615 | Gujarat Titans |
| 1068 | Deccan Chargers |
| 1414 | Kochi Tuskers Kerala |
| 1419 | Pune Warriors |
| 3604 | Rising Pune Supergiants |

## Step-by-Step Analysis

### Step 1 — Structure and Shape
- 16 rows × 2 columns — entire table is small enough to review in full
- 0 null values
- 0 exact duplicate rows

### Step 2 — team_id
- Unique: **yes** — valid primary key
- Not sequential: gaps exist (1, 2, 3, 4, 5, 6, 129, 134, 252, 494, 614, 615, 1068, 1414, 1419, 3604)
- **Non-sequential IDs are expected** — these are externally assigned identifiers, not auto-incremented

### Step 3 — team_name
- All 16 values are unique — no duplicate names
- No whitespace padding
- No empty strings

### Step 4 — Notable Findings

**Rising Pune Supergiant vs Rising Pune Supergiants:**
- team_id=4 → "Rising Pune Supergiant" (singular — 2017 official name)
- team_id=3604 → "Rising Pune Supergiants" (plural — 2016 official name)
- In match and ball data, team_id 4 was used for all 30 matches across both 2016 and 2017; team_id 3604 is an unused orphan ID in teams_data. Both entries are preserved in teams_data.

**Deccan Chargers (team_id=1068):**
- Present in teams_data as its own franchise (terminated in 2012; SRH was formed as a new franchise in 2013).
- In the clean dataset, team_id 2 for seasons <= 2012 is remapped to 1068 across matches and ball-by-ball deliveries, and the Deccan Chargers alias in team_aliases points to 1068.

### Step 5 — Whitespace Check
- All 16 team_name values passed the strip check — no changes required

## Cleaning Decisions

| Action | Result |
|---|---|
| Whitespace check on `team_name` | No padding found — no changes |

**No cleaning was applied.** The dataset is clean.

## Unresolved Issues

- **Deccan Chargers / SRH conflict** (described above) — requires SQL-layer resolution
- **Rising Pune dual IDs** — both retained; historical accuracy is preserved; SQL queries joining on team_id must account for this

---

---

# Cross-Dataset Summary

## Data Quality Matrix

| Issue Type | ball_by_ball | matches | players | aliases | teams |
|---|---|---|---|---|---|
| Null values | ✅ (all structural) | ✅ (all justified) | ⚠️ bowl_style, field_pos | ✅ None | ✅ None |
| Blank strings | ✅ None | ✅ None | 🔴 612 in field_pos; 54 in bowl_style | ✅ None | ✅ None |
| Exact duplicates | ✅ 0 | ✅ 0 | ✅ 0 | ✅ 0 | ✅ 0 |
| Invalid values | ✅ None confirmed | ✅ None | 🔴 3 rows (bowling text in field_pos) | ✅ None | ✅ None |
| Whitespace padding | ✅ None | ✅ None | ✅ None | ✅ None | ✅ None |
| Constant columns | — | 🟡 6 columns | — | — | — |
| Domain rule violations | ✅ None | ✅ None | — | — | — |
| Franchise ID conflict | — | — | — | ⚠️ Deccan Chargers | ⚠️ Deccan Chargers |

## Summary of All Cleaning Actions

| Dataset | Action | Rows Affected |
|---|---|---|
| ball_by_ball | Whitespace stripped from player/categorical columns | 0 (none found) |
| matches | match_date parsed to datetime | 1,212 (type conversion) |
| matches | Whitespace stripped from string columns | 0 (none found) |
| players | Blank strings → NaN in bowl_style | 54 |
| players | Blank strings → NaN in field_pos | 612 |
| players | Bowling text in field_pos set to NaN | 3 |
| aliases | Whitespace check only | 0 |
| teams | Whitespace check only | 0 |

**Total records deleted across all datasets: 0**  
**Original CSV files: all intact at original byte sizes**

## Issues Retained (To Be Resolved in SQL Layer)

| Issue | Dataset | Recommended Resolution |
|---|---|---|
| `city` nulls (51 rows) | matches | Derive from venue using a lookup table |
| `match_number` nulls (70 rows) | matches | Confirm playoff match numbering from IPL schedule |
| `batsman_type` / `bowler_type` gaps | ball_by_ball | Join with players table on batter/bowler name |
| `bowl_style` nulls (~73 rows) | players | Accept as unknown for non-bowlers |
| `field_pos` nulls (~700 rows) | players | Accept; most IPL players have no recorded position |
| 6 constant columns | matches | Drop at dimensional model stage |
| Deccan Chargers ID conflict | aliases + teams | Business decision: unify under SRH or keep separate |
| Rising Pune dual IDs | teams | Decide whether to merge for analysis or keep historical |

## Fitness for SQL Modelling

| Dataset | Suitability | Notes |
|---|---|---|
| `ball_by_ball_data` | ⭐⭐⭐⭐⭐ Excellent | Clean PK-ready grain; all Boolean flags reliable; run arithmetic validated |
| `ipl_matches_data` | ⭐⭐⭐⭐ Good | match_id is clean PK; justifiable nulls; 6 constant columns to remove |
| `players-data-updated` | ⭐⭐⭐⭐ Good | player_id is clean PK; style columns cleaned; field_pos sparse but not broken |
| `team_aliases` | ⭐⭐⭐⭐ Good | Clean lookup table; alias_id sequential PK; Deccan Chargers issue flagged |
| `teams_data` | ⭐⭐⭐⭐⭐ Excellent | Perfect 2-column reference; fully unique team_id; no data quality issues |
