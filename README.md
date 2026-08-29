# Global Sustainable Energy Dashboard (2000-2020)

Interactive dashboard on global sustainable energy indicators (electricity access, energy mix, renewable share, CO2 emissions) across 176 countries between 2000 and 2020. This is a portfolio project built as part of a transition into data analytics, focused on data visualization rather than data cleaning (see the other portfolio projects for that).

**[View the live dashboard on Tableau Public](https://public.tableau.com/app/profile/thibaut.r4710/viz/Book1_17849959495500/Dashboard1)**

![Dashboard overview](images/dashboard_overview.png)

## What it covers

- **Electricity Access Around the World (2000-2020)** — world map of electricity access (% of population) by country, with a year slider
- **Global Electricity Mix (2000-2020)** — fossil / nuclear / renewable electricity generation (TWh), stacked over time
- **Renewable Share Growth: Top 10 Countries (2000-2019)** — countries with the largest gain in renewable energy share, benchmarked against the world average
- **CO2 Emissions vs GDP per Capita** — CO2 emissions per capita against GDP per capita, one point per country, year slider synced with the map

## Data

Source: [Global Data on Sustainable Energy (2000-2020)](https://www.kaggle.com/datasets/anshtanwar/global-data-on-sustainable-energy) on Kaggle, by Ansh Tanwar.

File: `global_sustainable_energy.csv` (176 countries, 2000-2020, 21 columns).

## Data Preparation (done directly in Tableau Public, no separate cleaning step)

Two columns had significant missing values and were checked before deciding whether to keep or exclude them:

| Column | Missing | Decision | Why |
|---|---|---|---|
| `Financial flows to developing countries (US $)` | 57% (2,089 / 3,649 rows) | **Kept** | The missing values are not random: rows without a value have an average GDP per capita of $20,895, versus $3,521 for rows with a value. Missing means "not an aid recipient" (likely a high-income country), which is meaningful information, not a data quality issue. |
| `Renewables (% equivalent primary energy)` | 59% (2,137 / 3,649 rows) | **Excluded** | Missing values do not reliably mean "no renewable energy": 1,585 of the missing rows still have positive `Electricity from renewables (TWh)`. The gap looks like a coverage limitation of this specific metric's source, not a real-world signal, so it isn't safe to visualize as-is. |

Verification check for the `Financial flows` decision (average GDP per capita, split by whether the row has a value):

![Data check: average GDP per capita by presence of financial flow data](images/data_check_financial_flows.png)

Also fixed: the `Density (P/Km2)` column header contained a literal line break in the source file, renamed for clean display in Tableau.

## Key calculated fields

- **`Renewable Share 2000` / `Renewable Share 2020`** — isolates each country's renewable energy share for a single year (`IF [Year] = 2000 THEN ... END`)
- **`Renewable Share Change`** — uses a `{FIXED [Entity] : MAX(...)}` LOD expression to compare each country's 2000 and 2019 values on a single row, regardless of the active year filter (2019 was used instead of 2020, which has near-complete missing data for this field)
- **`World Avg Renewable Share Change`** — a table-wide `{FIXED : AVG(...)}` LOD expression, used as a reference line so the Top 10 chart can be benchmarked against the true world average rather than the average of the 10 countries shown
- **`Population (estimated)`** — `Density × Land Area`, used to derive a per-capita view (see limitation below)
- **`CO2 per Capita`** — `Value_co2_emissions_kt_by_country × 1000 / Population (estimated)`, converting kilotonnes to tonnes per person

## Known limitations

- **Estimated population.** The dataset has no direct population column, so population is approximated as `Density × Land Area`. Cross-checking against France's real population (~67.4M) shows a ~13% overestimation (76.6M), caused by a scope mismatch between the two source columns: `Land Area` appears to include overseas territories, while `Density` reflects metropolitan France only. `CO2 per Capita` and `Population (estimated)` should be read as directional, not exact.
- **2019 used instead of 2020 for renewable share growth**, due to near-complete missing data in 2020 for that specific field.

## Tech stack

- **Tableau Public**: data connection, calculated fields (LOD expressions), map, stacked area chart, bar chart with reference line, scatter plot, dashboard with synced filter actions
