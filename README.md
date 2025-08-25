[![Releases - download files](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases) https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases

# Iraq CO₂ Emissions & Energy Analysis — SQL Power BI Toolkit

📊 🌍 🔌

This README describes a data analysis project. It uses SQL, PostgreSQL, and Power BI. The project analyzes Iraq's CO₂ emissions and energy use from 1960 to 2024. The repo contains SQL scripts, a PostgreSQL schema, a Power BI report file, and documentation. Download the release file from the Releases page and execute the included SQL and PBIX files to run the project locally.

Badges
- Topics: co2-emissions • data-analysis-project • data-visualization • energy-analysis • interactive-dashboard • iraq • portfolio-project • postgresql • power-bi • real-world-data • sql
- Releases: [Download release files](https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases)

Cover image
![Iraq Emissions Banner](https://images.unsplash.com/photo-1542281286-9e0a16bb7366?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=600&q=60&w=1400)

Table of contents
- About
- Project goals
- Data sources
- Repo structure
- Quick start
- Releases (download and run)
- Environment and dependencies
- Database schema
- ETL and data prep
- Key SQL queries
- Power BI report
- Dashboard pages and visuals
- DAX measures and Power Query tips
- Analysis highlights and findings
- Validation and tests
- Performance and scaling
- Reproducibility and workflow
- Recommended next steps
- Contributing
- License
- Acknowledgements
- References and external links

About
This project uses real data. It explores trends in CO₂ emissions and energy use in Iraq. It covers fuels, electricity, and key sectors such as industry and transport. The analysis spans 1960 through projected values to 2024. The repo aims to show a clean workflow from raw data to an interactive dashboard.

Project goals
- Build a reproducible analytics pipeline using PostgreSQL and Power BI.
- Clean and join multi-source datasets on emissions and energy.
- Create an interactive dashboard for policy and technical audiences.
- Demonstrate SQL patterns for time-series energy data.
- Provide a shareable Power BI report (.pbix) and SQL artifacts.

Data sources
The project combines public datasets and synthetic enrichments for completeness. Core sources include:
- World Bank CO₂ and energy indicators (CO2 total, CO2 per capita, energy use)
- International Energy Agency (IEA) aggregated statistics (where available)
- National statistics from Iraq public portals and historical reports
- GADM and simple geospatial outlines for map visuals
- Synthetic projections and gap fills for years with missing official values

All raw sources and API calls are documented in the data folder. The release bundle contains copies that you can import if you prefer offline work.

Repo structure
- /data
  - raw/                  # raw source files (CSV, JSON)
  - staging/              # cleaned CSV used for loading
  - lookup/               # country codes, fuel categories
- /sql
  - schema.sql            # PostgreSQL schema and table DDL
  - load_data.sql         # COPY statements and load helpers
  - transforms.sql        # SQL transformations and views
  - sample_queries.sql    # example queries used for analysis
- /powerbi
  - iraq_emissions.pbix   # Power BI report file (in release)
  - readme_powerbi.md     # Power BI connection and refresh guide
- /notebooks
  - eda.ipynb             # optional exploratory analysis (Python)
- /docs
  - methodology.md        # detailed methods and assumptions
  - data_dictionary.md    # field definitions
  - performance.md        # tuning notes
- README.md
- LICENSE
- .gitignore

Quick start
1. Clone the repo:
   ```bash
   git clone https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis.git
   cd iraq-emissions-energy-analysis
   ```
2. Visit the Releases page and download the release bundle. The bundle contains the prebuilt database dumps and the Power BI file. You must download the release file and execute the included SQL and PBIX files to run the project locally: https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases
3. Follow the setup below to create the PostgreSQL database.
4. Load the staging CSV files using the provided load scripts.
5. Open the Power BI (.pbix) file in Power BI Desktop. Update data source credentials to connect to your local PostgreSQL database or import the provided local CSV files.

Releases (download and run)
- The Releases page contains the packaged artifacts for this project. Download the release file. The release includes:
  - Pre-built PostgreSQL dump of the cleaned database (pg_dump)
  - A ready-to-open Power BI report file (iraq_emissions.pbix)
  - A README in the release with step-by-step run commands
  - CSV copies of staging data
- After download, restore the dump or run the SQL scripts. Execute the PBIX file in Power BI Desktop.
- Releases link: [https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases](https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases)

Environment and dependencies
- PostgreSQL 12 or newer
- Power BI Desktop (latest stable)
- psql or pg_restore (for dump restore)
- Optional: Docker and docker-compose for a local Postgres container
- Optional: Python 3.8+ for notebooks and extra data prep

Suggested Docker quick setup
- docker-compose.yml (example)
  ```yaml
  version: '3.8'
  services:
    db:
      image: postgres:14
      restart: always
      environment:
        POSTGRES_USER: iraq_user
        POSTGRES_PASSWORD: iraq_pass
        POSTGRES_DB: iraq_emissions
      ports:
        - "5432:5432"
      volumes:
        - ./data:/data
  ```
- Start:
  ```bash
  docker-compose up -d
  ```
- Load data using psql or pg_restore once container starts.

Database schema
The schema focuses on time-series and dimension tables. It stores raw measures and derived metrics.

Tables
- dim_country
  - country_id (PK)
  - country_name
  - iso3
  - region
  - lat
  - lon
- dim_fuel
  - fuel_id (PK)
  - fuel_name
  - fuel_group (e.g., fossil, renewables)
- fact_energy
  - fact_id (PK)
  - country_id (FK -> dim_country)
  - fuel_id (FK -> dim_fuel)
  - year
  - energy_use_tj
  - energy_use_ktoe
  - electricity_consumption_gwh
  - sector                 # industry, transport, residential
- fact_emissions
  - emission_id (PK)
  - country_id (FK -> dim_country)
  - year
  - co2_total_mt
  - co2_per_capita_t
  - sector
- staging tables: staging_energy, staging_emissions, staging_population

Indexes and constraints
- Primary keys use serial/bigserial.
- Indexes on (country_id, year) for fast time-series queries.
- Partial indexes on fuel_group where needed for common filters.
- Foreign keys enforce referential integrity.

ETL and data prep
Steps
1. Place raw CSVs in /data/raw.
2. Run preprocessing scripts to standardize headers and units.
3. Load staging CSVs into staging tables using COPY commands.
4. Run transforms.sql to populate dimension and fact tables.
5. Run cleanup queries to check for outliers and data gaps.

Common transforms
- Unit conversion: ktoe -> TJ, GWh -> TJ, etc.
- Gap fill: linear interpolation for missing yearly values up to a 3-year gap.
- Aggregation: group fuel-level use into sectors and fuel groups.
- Normalization: per capita and intensity metrics.

Example load command
```sql
COPY staging_energy (country, iso3, year, fuel, value_ktoe, sector)
FROM '/path/to/data/raw/energy_iraq.csv'
DELIMITER ','
CSV HEADER;
```

Key SQL queries
This section lists queries used in analysis. They work against the normalized schema.

1. Yearly CO₂ trend for Iraq
```sql
SELECT year,
       SUM(co2_total_mt) AS total_co2_mt,
       AVG(co2_per_capita_t) AS per_capita_t
FROM fact_emissions
WHERE country_id = (SELECT country_id FROM dim_country WHERE iso3 = 'IRQ')
GROUP BY year
ORDER BY year;
```

2. Energy use by fuel group
```sql
SELECT f.fuel_group,
       e.year,
       SUM(e.energy_use_tj) AS energy_tj
FROM fact_energy e
JOIN dim_fuel f ON e.fuel_id = f.fuel_id
JOIN dim_country c ON e.country_id = c.country_id
WHERE c.iso3 = 'IRQ'
GROUP BY f.fuel_group, e.year
ORDER BY e.year, f.fuel_group;
```

3. CO₂ intensity: emissions per unit energy
```sql
SELECT e.year,
       SUM(co2_total_mt) AS co2_mt,
       SUM(energy_use_tj) AS energy_tj,
       CASE WHEN SUM(energy_use_tj) = 0 THEN NULL
            ELSE SUM(co2_total_mt) / SUM(energy_use_tj)
       END AS co2_per_tj
FROM fact_emissions em
JOIN fact_energy e ON em.country_id = e.country_id AND em.year = e.year
WHERE em.country_id = (SELECT country_id FROM dim_country WHERE iso3 = 'IRQ')
GROUP BY e.year
ORDER BY e.year;
```

4. Sectoral emissions share
```sql
SELECT sector,
       year,
       SUM(co2_total_mt) AS sector_co2_mt,
       SUM(co2_total_mt) / SUM(SUM(co2_total_mt)) OVER (PARTITION BY year) AS share
FROM fact_emissions
WHERE country_id = (SELECT country_id FROM dim_country WHERE iso3 = 'IRQ')
GROUP BY sector, year
ORDER BY year, share DESC;
```

5. Top years with largest year-on-year change
```sql
WITH yearly AS (
  SELECT year, SUM(co2_total_mt) AS total
  FROM fact_emissions
  WHERE country_id = (SELECT country_id FROM dim_country WHERE iso3 = 'IRQ')
  GROUP BY year
)
SELECT y1.year,
       y1.total,
       y1.total - COALESCE(y0.total, 0) AS delta
FROM yearly y1
LEFT JOIN yearly y0 ON y0.year = y1.year - 1
ORDER BY ABS(y1.total - COALESCE(y0.total, 0)) DESC
LIMIT 10;
```

Analytical patterns and tips
- Use window functions for moving averages and trends.
- Use CTEs to break complex transforms into readable steps.
- Use date ranges and partitions for fast aggregation by year.
- Keep raw staging data separate to simplify re-run of ETL.

Power BI report
The Power BI report (iraq_emissions.pbix) connects to PostgreSQL or local CSV files. It contains multiple report pages and interactive filters.

How to open
- Open Power BI Desktop.
- File > Open > select iraq_emissions.pbix from the release bundle.
- If the report uses a PostgreSQL source, update the credentials in Data Source Settings. Point to your local database or to provided CSV folder.

Data model
- Fact tables: fact_energy, fact_emissions
- Dimensions: dim_country, dim_fuel, date dimension built from year range
- Relationships: country_id, year, fuel_id
- Star schema optimized for slicing and filtering

Report pages and visuals
The report uses a set of pages that mirror the analysis workflow.

1. Overview
- KPI cards: total CO₂ (latest year), CO₂ per capita, total energy use
- Trend line: CO₂ total trend (1960–2024)
- Small multiples: emissions by sector
- Map: CO₂ intensity by governorate (if subnational data exists)

2. Energy Mix
- Stacked area chart: energy use by fuel group over time
- Bar chart: fuel share for selected year
- Waterfall: changes in energy by fuel group between two years

3. Sector Insights
- Treemap: sectoral energy and emissions share
- Sankey or chord style flow: energy source to sector where data maps allow

4. Emissions Drivers
- Scatter: GDP per capita (if present) vs CO₂ per capita
- Decomposition visual: changes due to population, intensity, and activity

5. Scenario and Projections
- Line charts showing historical vs projected values
- Toggle slicer to switch gap-fill method

6. Data Explorer
- Table visual with inline filters for year, fuel, and sector
- Export to CSV option

Visual choices
- Use area and stacked bar for composition and trend.
- Use line for trend and rate changes.
- Use card visuals for headline numbers.
- Use map with choropleth for spatial context.

DAX measures
Examples below show common measures used in the report.

1. Total CO₂ (latest year)
```dax
Total CO2 = SUM(fact_emissions[co2_total_mt])
```

2. CO₂ per capita (calculated)
```dax
CO2 per Capita = DIVIDE([Total CO2], SUM(dim_population[population]))
```

3. Year-over-Year change
```dax
YoY CO2 = 
VAR _prev = CALCULATE([Total CO2], DATEADD('Date'[Date], -1, YEAR))
RETURN DIVIDE([Total CO2] - _prev, _prev)
```

4. Moving average (3-year)
```dax
CO2 3yr MA = AVERAGEX(
  DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, YEAR),
  [Total CO2]
)
```

Power Query tips
- Use Query Parameters for database host, port, user, password, and database name.
- Use "Enable load" to disable unneeded staging queries.
- Use incremental refresh if dataset grows and you use Premium or Premium per user.
- Normalize units during ETL in PostgreSQL or in Power Query to reduce model size.

Dashboard interactions
- Slicers: year range, fuel group, sector
- Cross-filtering: clicking a fuel group filters charts across the page
- Drill-through: drill from country to governorate-level maps if subnational data exists

Analysis highlights and findings
This section describes sample findings that the report surfaces. These are examples drawn from the dataset and analysis steps. The exact numbers depend on the loaded data and the release file version.

Long-term trend
- CO₂ emissions show a long-term rise from 1960 to the early 1980s. The rise reflects the expansion of oil production and energy use.
- Conflict periods show drops in energy use and emissions in some years.
- From 2003 onward, data shows a recovery and then a growth trend to 2014. The trend moderates after 2014, with variations tied to production, infrastructure, and electricity supply.

Fuel mix
- Oil and gas dominate primary energy use. Natural gas use rises as gas-to-power projects and field development proceed.
- Electricity demand grows in urban areas. Grid losses and generation fuel mix influence emission intensity.

Per capita metrics
- Per capita CO₂ grows in years with strong energy access and economic recovery.
- Improvements in energy efficiency in some periods reduce intensity per unit of GDP or per kWh in specific sub-sectors.

Sector drivers
- Industry and power generation account for a large share of total emissions.
- Transport shares grow with vehicle fleet expansion and freight activity increases.

Policy-relevant metrics
- CO₂ per kWh of electricity provides a simple intensity measure useful for planning decarbonization.
- Emissions per capita and per unit energy highlight where efficiency gains matter most.

Validation and tests
- Cross-check totals against World Bank aggregates for overlapping years.
- Run sanity checks: no negative energy use except flagged corrections, year ranges match, and totals aggregate correctly.
- Use null checks to alert for missing years or country mismatches.

Sample validation queries
1. Check missing years in emissions
```sql
SELECT year
FROM generate_series(1960, 2024) AS year
LEFT JOIN (
  SELECT DISTINCT year FROM fact_emissions WHERE country_id = (SELECT country_id FROM dim_country WHERE iso3 = 'IRQ')
) f ON year = f.year
WHERE f.year IS NULL;
```

2. Check negative values
```sql
SELECT *
FROM fact_energy
WHERE energy_use_tj < 0 OR energy_use_ktoe < 0;
```

Performance and scaling
- Index country_id and year columns.
- Use partitioning by year for large fact tables if dataset grows beyond millions of rows.
- Use materialized views for expensive aggregates used in dashboards.
- Use read replicas for heavy dashboard loads.

Example: Create a materialized view for yearly aggregation
```sql
CREATE MATERIALIZED VIEW mv_yearly_energy AS
SELECT country_id, year, SUM(energy_use_tj) AS total_energy_tj
FROM fact_energy
GROUP BY country_id, year;
```
Refresh daily or on demand:
```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_yearly_energy;
```

Reproducibility and workflow
- Keep raw data in /data/raw. Do not edit raw files.
- Use SQL scripts in /sql to run ETL in a repeatable order.
- Keep versions of PBIX in releases if model changes break reports.
- Use semantic versioning for releases: v1.0.0, v1.1.0, etc.

Suggested workflow
1. Update or add raw data to /data/raw.
2. Run preprocessing scripts to normalize units.
3. Load staging files to PostgreSQL.
4. Run transforms.sql to refresh facts and dimensions.
5. Refresh Power BI data model or open the PBIX and refresh.

Which files to run
- schema.sql sets up tables and constraints.
- load_data.sql loads CSV files into staging tables.
- transforms.sql builds final tables and views.
- sample_queries.sql contains the queries used for charts.

If you downloaded the release bundle, it contains a README that lists the exact files to run and the execution order.

Recommended next steps
- Add subnational data where available to enhance map insights.
- Add GDP and sector activity data for decomposition analysis.
- Add scenario modeling for energy transition paths.
- Add automated tests that check totals against known benchmarks.
- Publish the dashboard to Power BI Service and set up scheduled refresh if using cloud sources.

Contributing
- Use issues to propose enhancements or report bugs.
- Use pull requests for code or documentation changes.
- Follow the contribution checklist:
  - Run linter and SQL formatter.
  - Add tests for new SQL transforms.
  - Update data dictionary for any schema changes.

Commit style
- Use clear commit messages. Use present tense. Use short summaries and longer descriptions when needed.

Code of conduct
Respectful collaboration only. Use GitHub issues and PR comments in a civil tone.

License
This project uses the MIT License. See LICENSE file for details.

Acknowledgements
- Data from World Bank and public energy reports.
- Map imagery courtesy of open sources.
- Design inspiration from standard energy dashboards.

References and external links
- World Bank Data: https://data.worldbank.org
- IEA: https://www.iea.org
- Power BI documentation: https://docs.microsoft.com/power-bi
- PostgreSQL documentation: https://www.postgresql.org/docs

Assets and images used
- Banner image: Unsplash (royalty free)
- Icons: Font Awesome and built-in Power BI visuals

If the Releases link fails
If the release link shows an error or you cannot download the release, check the Releases section on the repository page for artifacts and manual download options: https://github.com/Muhtasim-Rashid-Mahin/iraq-emissions-energy-analysis/releases

Contact and credits
- Repo owner: Muhtasim Rashid Mahin
- For questions, open an issue on the repository or use the contact information on the GitHub profile.

Appendix A — Full list of SQL scripts and order to run
1. schema.sql
   - Create schemas
   - Create dim_country, dim_fuel, staging tables
2. load_data.sql
   - COPY statements for staging CSVs
3. transforms.sql
   - Populate dim tables
   - Build fact tables from staging
   - Create indexes and constraints
4. sample_queries.sql
   - Run queries for validation and figures used in report

Appendix B — Data dictionary (sample fields)
- country_id: integer primary key
- iso3: 3-letter country code
- year: integer (e.g., 2020)
- energy_use_tj: numeric, energy in terajoules
- energy_use_ktoe: numeric, energy in kilotons of oil equivalent
- electricity_consumption_gwh: numeric, electricity in GWh
- co2_total_mt: numeric, CO₂ in million tonnes
- co2_per_capita_t: numeric, tonnes per person

Appendix C — Example ETL timeline and commands
1. Create DB and user
```bash
psql -U postgres -c "CREATE USER iraq_user WITH PASSWORD 'iraq_pass';"
psql -U postgres -c "CREATE DATABASE iraq_emissions OWNER iraq_user;"
```
2. Run schema
```bash
psql -U iraq_user -d iraq_emissions -f sql/schema.sql
```
3. Load staging data
```bash
psql -U iraq_user -d iraq_emissions -f sql/load_data.sql
```
4. Run transforms
```bash
psql -U iraq_user -d iraq_emissions -f sql/transforms.sql
```
5. Validate
```bash
psql -U iraq_user -d iraq_emissions -f sql/sample_queries.sql
```

Appendix D — Power BI refresh and gateway
- For local refresh use Power BI Desktop.
- For scheduled refresh, publish to Power BI Service. Use an on-premises data gateway to connect to PostgreSQL in your environment.
- Use data source parameters to avoid hard-coding credentials.

Appendix E — Example visualization notes
- Use a dark-neutral theme for maps and line charts.
- Use three consistent colors for fuel groups: fossil (orange), gas (blue), renewables (green).
- Use year slider at the top for global context.
- Add tooltips with exact numbers and percent shares for transparency.

Repository topics (for discovery)
- co2-emissions
- data-analysis-project
- data-visualization
- energy-analysis
- interactive-dashboard
- iraq
- portfolio-project
- postgresql
- power-bi
- real-world-data
- sql

End of file