# Muhammad Zia Ul Haq — Data Analytics Portfolio

Senior BI & Insights Analyst | Power BI & Microsoft Fabric · Dubai, UAE

I build analytics end to end: from ingesting raw data to the model and report that decision-makers use. 12+ years in data operations and analytics.

---

## Featured projects

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

## Power BI & DAX patterns

Focused write-ups of modeling patterns for common business problems.

| Pattern | Problem | Key techniques |
|---|---|---|
| [Account Rotation (3+3)](./account-rotation-3plus3) | B2B account health, where recency alone misclassifies accounts | `SWITCH` classification, inactive relationships, weighted scoring, dynamic RLS |
| [Recurring-Demand Forecasting](./bunker-opportunities-overview) | Predicting when an asset next needs servicing | Baselines learned from service history, calculated tables, tiered alerting |
| [Cash Utilization Tracker](./cash-utilization-tracker) | Working capital: how much is left, and when it runs out | Running totals, `TREATAS`, forward projection from payment behavior |

## BI governance

[**BI Governance & Standards**](./bi-governance): a framework for running Power BI across a team, covering modeling and DAX standards, row-level security, onboarding, a maturity model and success metrics.

---

## Skills

**Microsoft Fabric:** Data Factory pipelines, Lakehouse, PySpark, Delta Lake, Direct Lake, ML experiments (MLflow)
**Power BI:** DAX, Power Query, data modeling, RLS, Tabular Editor
**Python & statistics:** pandas, scikit-learn, panel regression, time-series forecasting, SHAP
**Also:** SQL, Excel, SharePoint, Power Automate, Tableau

## Certifications

- Microsoft Certified: Power BI Data Analyst Associate (PL-300)
- Salesforce Certified Tableau Desktop Foundations
- In progress: Microsoft Certified: Fabric Analytics Engineer Associate (DP-600)

## Contact

[LinkedIn](https://www.linkedin.com/in/mziamalik) · zulhaq@gmail.com
