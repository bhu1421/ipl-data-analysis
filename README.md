# IPL Performance Analytics & Insights

An end-to-end IPL cricket analytics project using **Python, Pandas, SQL,
and Power BI**. It combines exploratory data analysis (EDA),
data-quality validation, historical franchise standardization,
dimensional modeling, and an interactive dashboard.

## Live Power BI Dashboard

Explore the interactive IPL Performance Analytics dashboard:

[**View the Live Power BI Report**](https://app.powerbi.com/groups/0a334a76-91ec-49cf-93ab-6def0cdbad56/reports/92e63f5f-ef3c-472c-8740-bd9226e49ae6/b76132fab069ba68bd9b?experience=power-bi)

## Project at a Glance

  ------------------------------------------------------------------------
  Dataset                                    Records Purpose
  --------------------- ---------------------------- ---------------------
  Ball-by-ball                               288,226 Delivery-level
  deliveries                                         batting, bowling,
                                                     extras, boundaries,
                                                     dismissals

  IPL matches                                  1,212 Match results, teams,
                                                     dates, venues, toss
                                                     information

  Player reference data                          799 Player attributes and
                                                     playing styles

  Team aliases                                    46 Alternate team names
                                                     and franchise
                                                     mappings

  Team reference data                             16 Franchise reference
                                                     data
  ------------------------------------------------------------------------

The available data spans **2008--2026**, with 2026 represented as a
partial season. Counts reflect the project datasets and may differ from
published totals.

## Objectives

-   Compare team performance and season-level trends.
-   Explore batting output, boundary scoring, and player rankings.
-   Evaluate bowler-credited wickets and bowling efficiency.
-   Validate source data and standardize historical team identities.
-   Build a dimensional model and interactive Power BI report.

## Tools and Technologies

-   **Python / Pandas:** data inspection, cleaning, transformation, EDA
-   **Jupyter Notebook:** documented analysis workflow
-   **SQL / MySQL:** relational storage and dimensional modeling
-   **Power BI:** interactive reporting and visualization
-   **DAX:** analytical measures and context-aware calculations
-   **Git / GitHub:** version control and documentation

## Workflow

``` text
Five source CSV files
        |
        v
Exploratory Data Analysis (Python / Pandas)
        |
        v
Data-quality checks and cleaning
        |
        v
Standardized cleaned datasets
        |
        v
Dimensional model
  ├── dim_teams
  ├── dim_players
  ├── dim_venues
  ├── dim_date
  ├── fact_matches
  └── fact_deliveries
        |
        v
Power BI data model and DAX measures
        |
        v
Four-page interactive dashboard
```

The EDA notebooks are for investigation and validation. Cleaning and
modeling steps produce the downstream analytical tables where those
outputs are available.

## Data Quality and Modeling

### Validation checks

The documented analysis includes checks for row counts, data types,
identifiers, missing values, whitespace-only strings, category
consistency, team and venue labels, run arithmetic, dismissal fields,
and legal-delivery flags.

One validation found **zero arithmetic mismatches across 288,226
delivery records** for:

`batter_runs + extras = total_runs`

This confirms that arithmetic rule for the checked records; it does not
prove that every source field is error-free.

### Historical franchise identity

Deccan Chargers and Sunrisers Hyderabad are treated as separate
franchises because they represent different teams in different periods.
A documented historical team-ID remapping prevents their records from
being combined in team-level analysis.

### Dimensional model

The documented Power BI model contains two fact tables and four
dimensions:

-   `fact_deliveries` --- delivery-level events and measures
-   `fact_matches` --- match-level results and attributes
-   `dim_teams` --- standardized franchise details
-   `dim_players` --- player attributes
-   `dim_venues` --- venue and city information
-   `dim_date` --- date attributes for time-based analysis

The fact tables have different grains. Measures should use the
appropriate fact table to avoid double-counting match-level and
delivery-level data.

### Partial 2026 season

For fair year-over-year comparisons, the project's standard comparison
period is **2008--2025**, unless a visual explicitly identifies that it
includes partial 2026 data.

## Power BI Dashboard

### 1. Executive Overview

High-level IPL activity and key performance indicators.

### 2. Team & Season Performance

Historical team comparisons, win performance, season trends, and
consistency analysis.

### 3. Batting Analysis

Run scoring, boundaries, batting trends, and player-level batting
performance.

### 4. Bowling Analysis

Wicket-taking and bowling efficiency, including economy rate, bowling
average, and bowling strike rate.

**Visual design:** dark navy stadium-inspired background, cyan/blue
accents, white typography, orange comparison elements, KPI cards, and
left-side page navigation.

> Add screenshots or a GIF of the actual Power BI report before
> publishing. Do not present screenshots of planned pages as completed
> features.

## Selected Findings

The following are documented descriptive findings. Results can change
with filters and metric definitions.

-   **Six-hitting increased:** season sixes rose from **623 in 2008 to
    1,302 in 2025**, approximately a **109% increase**.
-   **Boundary contribution in 2025:** fours and sixes accounted for
    approximately **66.6% of batter runs** in the documented analysis.
-   **Historical team comparison:** Mumbai Indians recorded 153 wins in
    285 matches, while Chennai Super Kings recorded 145 wins in 260
    matches in the documented all-season analysis.
-   **Dismissal profile:** caught dismissals represented approximately
    **63.1% of recorded dismissals** in the documented EDA.

These findings describe patterns; they do not establish that any single
factor causes a team to win. Raw seasonal totals should be interpreted
in the context of the number of matches played.

## Repository Structure

The exact files committed to GitHub may vary. The core project is
organized around source CSVs, EDA notebooks, reports, and data outputs.

``` text
ipl-data-analysis/
├── data/
│   ├── cleaned/                  # Cleaned datasets, where included
│   └── powerbi_tables/           # Dimensions and fact tables, where included
├── notebooks/
│   └── eda/
│       ├── 01_ball_by_ball_eda.ipynb
│       ├── 02_ipl_matches_eda.ipynb
│       ├── 03_players_eda.ipynb
│       ├── 04_team_aliases_eda.ipynb
│       ├── 05_teams_eda.ipynb
│       └── 06_sql_database_setup.ipynb
├── ball_by_ball_data.csv
├── ipl_matches_data.csv
├── players-data-updated.csv
├── team_aliases.csv
├── teams_data.csv
├── ipl_eda_full_report.md
├── project_report.md
├── .gitignore
└── README.md
```

Some generated files shown above may exist locally rather than in
GitHub. Check the repository before relying on a specific path.

## Getting Started

### Prerequisites

-   Python 3
-   Jupyter Notebook or JupyterLab
-   Pandas and the libraries used by the notebooks
-   MySQL, if running the database setup notebook
-   Power BI Desktop, if the `.pbix` report is available

### 1. Clone the repository

``` bash
git clone https://github.com/bhu1421/ipl-data-analysis.git
cd ipl-data-analysis
```

### 2. Create a virtual environment

**Windows**

``` bash
python -m venv .venv
.venv\Scripts\activate
```

**macOS / Linux**

``` bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install common analysis packages

``` bash
python -m pip install pandas numpy matplotlib seaborn jupyter sqlalchemy pymysql
```

Install packages required by the notebooks you run. If the repository
includes a `requirements.txt`, use
`python -m pip install -r requirements.txt`.

### 4. Run the EDA notebooks

``` bash
jupyter notebook
```

Open `notebooks/eda/` and run the notebooks as needed. Check each
notebook's data paths against the location of the CSV files in your
checkout.

### 5. Set up SQL (optional)

Open `06_sql_database_setup.ipynb` and follow its database setup cells.
Configure credentials locally. **Never commit passwords, connection
strings, or `.env` files containing secrets.** The database setup may
depend on cleaned outputs created by the project's cleaning step.

### 6. Open Power BI (optional)

Open the `.pbix` file in Power BI Desktop if it is available locally. If
it is not included in the repository, the report file must be obtained
separately. Data-source paths and credentials may need to be updated.

## Important Metric Definitions

-   **Batter runs:** runs credited to the batter, excluding extras.
-   **Fours / sixes:** counts of deliveries marked as batter-hit
    boundaries in the prepared data.
-   **Bowler-credited wickets:** wickets attributed to the bowler;
    run-outs and other non-bowler-credited dismissals should be
    excluded.
-   **Economy rate:** runs conceded × 6 ÷ legal balls bowled. The
    runs-conceded calculation must handle byes, leg-byes, and other
    extras consistently.
-   **Bowling average:** runs conceded ÷ bowler-credited wickets.
-   **Bowling strike rate:** legal balls bowled ÷ bowler-credited
    wickets.
-   **Win percentage:** matches won ÷ matches played. Treatment of
    abandoned, tied, no-result, and playoff matches should be
    consistent.
-   **Season comparisons:** per-match or per-innings metrics can be more
    informative than raw totals when season lengths differ.

Validate the report's DAX measures against these definitions before
interpreting the results.

## Limitations and Future Enhancements

### Current limitations

-   The 2026 season is partial.
-   Historical team and venue labels require standardization.
-   Ball-by-ball metrics depend on source-data conventions.
-   Findings are descriptive, not causal.
-   The GitHub repository may not include the Power BI `.pbix` file or
    all generated tables.

### Future enhancements

-   Build a dedicated Toss Analysis page.
-   Build a dedicated Venue Analysis page.
-   Add per-match and per-innings comparisons to account for season
    length.
-   Include dashboard screenshots and a short walkthrough.
-   Add a reproducible `requirements.txt` and automated data-quality
    tests.

Toss Analysis and Venue Analysis are proposed enhancements, not
implemented dashboard pages.

## Author

**Bhuvan Patil**

This project demonstrates practical work in data cleaning, exploratory
analysis, SQL, dimensional modeling, DAX, and Power BI dashboard
development.
