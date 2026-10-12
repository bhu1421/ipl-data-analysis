# IPL Analytics --- Data Pipeline & Power BI Dashboard

An end-to-end data analytics project for Indian Premier League (IPL)
cricket data, covering **2008--2026**. The project includes exploratory
data analysis (EDA), data cleaning, a MySQL star schema, and datasets
prepared for Power BI.

## Project Overview

-   **Seasons:** 2008--2026 *(2026 is a partial season in this dataset)*
-   **Matches:** 1,212
-   **Ball-by-ball deliveries:** 288,226
-   **Players:** 799
-   **Teams:** 16
-   **Canonical venues:** 36

## Repository Structure

``` text
ipl-data-analysis/
├── data/                     # Data files, if included in the repository
├── notebooks/
│   └── eda/
│       ├── 01_ball_by_ball_eda.ipynb
│       ├── 02_ipl_matches_eda.ipynb
│       ├── 03_players_eda.ipynb
│       ├── 04_team_aliases_eda.ipynb
│       ├── 05_teams_eda.ipynb
│       └── 06_sql_database_setup.ipynb
├── ball_by_ball_data.csv     # Raw ball-by-ball data, if stored at the root
├── ipl_matches_data.csv      # Raw match data, if stored at the root
├── players-data-updated.csv  # Raw player data, if stored at the root
├── team_aliases.csv          # Team aliases, if stored at the root
├── teams_data.csv            # Raw team data, if stored at the root
├── .gitignore
└── README.md
```

## Data Sources

The project uses five source CSV files:

-   `ball_by_ball_data.csv` --- delivery-level records
-   `ipl_matches_data.csv` --- match-level records
-   `players-data-updated.csv` --- player details
-   `team_aliases.csv` --- alternate team names and franchise mappings
-   `teams_data.csv` --- team reference data

## Workflow

### 1. Exploratory Data Analysis

The first five notebooks explore the raw datasets independently:

  -----------------------------------------------------------------------
  Notebook                            Focus
  ----------------------------------- -----------------------------------
  `01_ball_by_ball_eda.ipynb`         Delivery patterns, extras, and
                                      wickets

  `02_ipl_matches_eda.ipynb`          Season trends, venues, and match
                                      results

  `03_players_eda.ipynb`              Player roles and batting/bowling
                                      styles

  `04_team_aliases_eda.ipynb`         Team-name variants and franchise
                                      history

  `05_teams_eda.ipynb`                Team reference data and franchise
                                      overview
  -----------------------------------------------------------------------

### 2. Data Cleaning

The cleaning stage standardizes and validates source data before it is
used downstream. The documented transformations include:

-   Separating Deccan Chargers from Sunrisers Hyderabad across their
    respective seasons.
-   Standardizing venue names and city values.
-   Adding match-status and playoff indicators.
-   Identifying legal deliveries and numbering legal balls.
-   Standardizing player attributes and converting whitespace-only
    values to missing values.

The original source files should remain unchanged; cleaned outputs are
generated separately.

### 3. Star Schema and Power BI

The `06_sql_database_setup.ipynb` notebook builds a dimensional model
for analysis. The documented model contains:

**Dimension tables** - `dim_teams` - `dim_players` - `dim_venues` -
`dim_date`

**Fact tables** - `fact_matches` --- one row per match -
`fact_deliveries` --- one row per delivery

The resulting tables are intended for Power BI analysis. Measures such
as batting averages, strike rates, and bowling economy rates can be
created in DAX.

## Using the Project

1.  Clone or download this repository.
2.  Install the Python libraries required by the notebooks.
3.  Open the EDA notebooks in Jupyter Notebook or VS Code.
4.  Run the notebooks in order as needed.
5.  For the database setup notebook, configure your local MySQL
    connection before running database-related cells.
6.  Load the prepared dimensional and fact tables into Power BI and
    create the required model relationships.

## Configuration and Security

-   Store database credentials in a local `.env` file.
-   **Never commit `.env` files, passwords, or other credentials to
    GitHub.**
-   Use `.env.example` to document required environment-variable names
    without including secret values.

## Notes

-   The EDA notebooks are intended for exploration and do not themselves
    form the data-cleaning pipeline.
-   Dataset row counts and coverage describe the version documented for
    this project; verify them if the source data changes.
