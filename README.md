# Zia Malik - Data & Analytics Portfolio

Senior Data Analyst | Power BI · Tableau · Microsoft Fabric · Databricks · Snowflake · dbt · BigQuery · DuckDB · Dubai, UAE

I build analytics end to end: from ingesting raw data to the model and report that decision-makers use. 12+ years in data operations and analytics. Every project below uses public data, so the code, data and reports can all be checked.

---

## Featured projects

### [The 2026 Oil Supply Shock](https://github.com/zulhaq-analytics/oil-shock-2026-databricks)

[![Oil shock report](https://raw.githubusercontent.com/zulhaq-analytics/oil-shock-2026-databricks/main/images/01-the-shock.png)](https://github.com/zulhaq-analytics/oil-shock-2026-databricks)

A twice-daily Databricks pipeline over about ten public sources (EIA, JODI, IMF PortWatch, FRED, Singapore MPA, CFTC): Python loaders into bronze tables, a tested dbt project into silver and gold, MLflow-tracked models (a Hormuz scenario calculator, a weekly US crude stock nowcast and an 80% range around EIA's outlook), and a nine-page Power BI report that refreshes itself after each run. My own Brent forecast had to beat a simple no-change forecast under a rule set before seeing results; it didn't, so the report uses EIA's outlook instead.

**Finding:** with Hormuz tanker traffic down to about 2% of 2025 levels, Gulf crude output fell about 25% and the rest of the world added only about 1 mb/d. Brent spot rose about 77%, and diesel and jet fuel even more, but in today's dollars Brent is still below its 2008 peak.

🔗 **[View project →](https://github.com/zulhaq-analytics/oil-shock-2026-databricks)** · **[Open the live report →](https://app.powerbi.com/view?r=eyJrIjoiYTRiMzkxZTAtZTc2ZS00N2M1LWE1ODQtYzBmNDJmNzgwNWI5IiwidCI6ImEyYjYxNTdiLWZlM2ItNGRlZi05OTAzLTc4YTRlMmU5NTNhYiJ9)**

---

### [Europe's Power Desk: Where, When and Why Electricity Gets Cheap](https://github.com/zulhaq-analytics/european-power-prices-databricks)

[![European power dashboard](https://raw.githubusercontent.com/zulhaq-analytics/european-power-prices-databricks/main/images/01-market-overview.png)](https://github.com/zulhaq-analytics/european-power-prices-databricks)

A daily Databricks pipeline over seven years of hourly ENTSO-E data for 41 European bidding zones (about 8 GB of raw XML, 47 million rows): a Python API client with retries, rate limiting and per-zone watermarks, a PySpark XML parser that converts every hour to the zone's local time, idempotent Delta Lake MERGE loads, Spark SQL gold tables, and a seven-page Tableau Public dashboard that refreshes every day through Google Sheets. Germany-Luxembourg's yearly averages match published German figures to the cent.

**Finding:** prices across one market are far apart: in 2025, Norway North averaged €9/MWh and Germany-Luxembourg €89. Negative prices are now routine, with 569 hours below zero in Germany-Luxembourg (6.5% of the year, up from 211 in 2019), and solar has made midday power 54% cheaper than the evening peak.

🔗 **[View project →](https://github.com/zulhaq-analytics/european-power-prices-databricks)** · **[Open the live dashboard →](https://public.tableau.com/views/EuropesPowerDesk/MarketOverview)**

---

### [Customer Lookbook: Who Buys, Who Stays and Who's Leaving at H&M](https://github.com/zulhaq-analytics/fashion-customer-analytics-bigquery)

[![H&M customer report](https://raw.githubusercontent.com/zulhaq-analytics/fashion-customer-analytics-bigquery/main/images/01-who-they-are.png)](https://github.com/zulhaq-analytics/fashion-customer-analytics-bigquery)

A Google Cloud build over 31.8 million real H&M purchases by 1.37 million customers: a BigQuery warehouse, a tested Dataform project (staging, star schema, cohorts, RFM groups and report views, rebuilt monthly on a schedule), two BigQuery ML models that predict who will stop buying and what they will spend, and a seven-page Power BI report with a what-if retention simulator.

**Finding:** the best customers (19% of buyers) bring in 61% of sales, and about half of last year's customers are likely to stop buying in the next 3 months. Targeting the top 10% by sales at risk reaches 32% of it, so a small, focused retention campaign goes a long way.

🔗 **[View project →](https://github.com/zulhaq-analytics/fashion-customer-analytics-bigquery)** · **[Open the live report →](https://app.powerbi.com/view?r=eyJrIjoiNTM2ZjJiMWMtZjQwMy00NDgwLTg0ZjQtN2NiOWNkNTM4ZGJiIiwidCI6ImEyYjYxNTdiLWZlM2ItNGRlZi05OTAzLTc4YTRlMmU5NTNhYiJ9)**

---

### [Dubai Property Prospectus](https://github.com/zulhaq-analytics/dubai-real-estate-snowflake-dbt)

[![Dubai property report](https://raw.githubusercontent.com/zulhaq-analytics/dubai-real-estate-snowflake-dbt/main/images/the-market.png)](https://github.com/zulhaq-analytics/dubai-real-estate-snowflake-dbt)

A daily pipeline over 15 Dubai Land Department open datasets (15.4 million rows): a Python loader that pulls new records from the DLD API every morning, a DuckDB warehouse, a tested dbt project (staging, intermediate and mart layers, 100+ tests, snapshots) that runs on both DuckDB and Snowflake, a 12-month forecast in Python, and an 11-page Power BI report on prices, rental yields, developers, sales speed and financing that refreshes itself daily through a gateway. The first version ran on Snowflake with role-based security, key-pair service accounts and Snowflake ML. After the move, 13 of 17 reconciliation checks matched exactly and the rest differed by 4 sales that DLD had re-dated.

**Finding:** 2025 was a record year, with 214,532 sales worth AED 662.7bn, up 19.7%. Ready homes yielded 6.4% gross and 5.4% net of service charges, but new launches are selling more slowly: 77% of units sold in the first year, down from 96% for 2022 launches.

🔗 **[View project →](https://github.com/zulhaq-analytics/dubai-real-estate-snowflake-dbt)** · **[Open the live report →](https://app.powerbi.com/view?r=eyJrIjoiN2FkZTRmYTQtOGY3ZC00YjA4LTg5ZTktZTY0MjY3NjVkMTc0IiwidCI6ImEyYjYxNTdiLWZlM2ItNGRlZi05OTAzLTc4YTRlMmU5NTNhYiJ9)**

---

### [U.S. Tariffs & Trade: Cut or Rerouted?](https://github.com/zulhaq-analytics/us-tariffs-trade-fabric)

[![Tariffs and trade report](https://raw.githubusercontent.com/zulhaq-analytics/us-tariffs-trade-fabric/main/docs/screenshots/01_overview.png)](https://github.com/zulhaq-analytics/us-tariffs-trade-fabric)

An end-to-end Microsoft Fabric solution over 23.8 million U.S. Census import records (2010–2026): a resumable API ingestion pipeline, a PySpark medallion Lakehouse reconciled to the dollar, a fixed-effects panel regression, 12-month forecasts with a plausibility guardrail, a source-shift prediction model, and a six-page Power BI report.

**Finding:** the 2025 tariffs did both. Rerouting held roughly steady against the 2018–19 trade war (23.9% → 26.4% of China-exposed import value), but outright cutting more than doubled (13.4% → 28.8%). China fell from about 22% of U.S. imports to about 8%, and when the Supreme Court struck the tariffs down, only 3.2% of that value moved back.

🔗 **[View project →](https://github.com/zulhaq-analytics/us-tariffs-trade-fabric)**

---

### [U.S. Flight Performance: Pre- vs Post-COVID](https://github.com/zulhaq-analytics/flight-performance-fabric)

[![Flight performance report](https://raw.githubusercontent.com/zulhaq-analytics/flight-performance-fabric/main/docs/screenshots/01_report_overview.png)](https://github.com/zulhaq-analytics/flight-performance-fabric)

An automated Microsoft Fabric solution over 49.3 million U.S. government flight records: a Data Factory pipeline, a PySpark medallion Lakehouse with quality-gate tests, a Direct Lake semantic model, and a five-page Power BI report.

**Finding:** headline totals show volume 5.3% below 2019, but that's because six carriers left the reporting set. The 12 airlines that reported throughout now fly more than before COVID, while on-time performance fell from 81.6% to 78.3%.

🔗 **[View project →](https://github.com/zulhaq-analytics/flight-performance-fabric)**

---

## Skills

**Cloud data platforms:** Databricks (Unity Catalog, Delta Lake, PySpark, Spark SQL, scheduled jobs, MLflow, Git folders), Microsoft Fabric (Data Factory, Lakehouse, PySpark, Delta Lake, Direct Lake), Snowflake (RBAC, key-pair authentication, resource monitors, Snowflake ML), Google BigQuery (BigQuery ML, scheduled workflows), DuckDB (local warehouse, Parquet export)

**Transformation:** dbt (layered modeling, tests, macros, snapshots, scheduled jobs), Dataform, SQL

**Power BI:** DAX, Power Query, data modeling, RLS, Tabular Editor, gateway scheduled refresh

**Tableau:** Tableau Public dashboards with daily data sync, parameter-driven pages, map flow lines

**Python & statistics:** pandas, REST API ingestion, scikit-learn, panel regression, time-series forecasting, SHAP

**Also:** Git and GitHub, Excel, SharePoint, Power Automate, Google Sheets API, Claude (AI-assisted development)

## Certifications

- Microsoft Certified: Power BI Data Analyst Associate (PL-300)
- Salesforce Certified Tableau Desktop Foundations
- In progress: Microsoft Certified: Fabric Analytics Engineer Associate (DP-600)

## Contact

[LinkedIn](https://www.linkedin.com/in/mziamalik) · zulhaq@gmail.com
