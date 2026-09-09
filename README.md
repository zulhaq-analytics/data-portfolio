# 👋 Hi, I'm Zia — Data Strategy & Insights Analyst | Power BI Developer

Welcome to my analytics portfolio.

I specialize in transforming raw data into clear, actionable insights using **Power BI, DAX, Power Query, and SQL**. My work focuses on solving real business problems across **Trading, Finance, Operations, and CRM** by building scalable, automated, and data-driven reporting solutions.

This repository documents selected **modeling patterns** from that work — the semantic model, the DAX, and the design reasoning behind each. Each write-up is a reference implementation you can adapt, not a description of any particular deployment.

---

## ⚙️ Skills & Tools

<p align="left">
<img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=Power%20BI&logoColor=black"/>
<img src="https://img.shields.io/badge/Microsoft%20Fabric-0078D4?style=for-the-badge&logo=microsoft&logoColor=white"/>
<img src="https://img.shields.io/badge/DAX-0A0A0A?style=for-the-badge&logo=Microsoft&logoColor=white"/>
<img src="https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white"/>
<img src="https://img.shields.io/badge/SQL-336791?style=for-the-badge&logo=postgresql&logoColor=white"/>
<img src="https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white"/>
<img src="https://img.shields.io/badge/SharePoint-0078D4?style=for-the-badge&logo=microsoft-sharepoint&logoColor=white"/>
<img src="https://img.shields.io/badge/Power%20Automate-0066FF?style=for-the-badge&logo=power-automate&logoColor=white"/>
<img src="https://img.shields.io/badge/Tableau-E97627?style=for-the-badge&logo=tableau&logoColor=white"/>
</p>

**Certifications:** Microsoft Certified: Power BI Data Analyst Associate (PL-300) · Salesforce Certified Tableau Desktop Foundations · Microsoft DP-600 (Fabric Analytics Engineer) — in progress

---

## 💡 My Approach to Analytics

I build BI solutions that balance **technical rigor with business usability**. Dashboards prioritize actionable insights, not data overload, and models are designed for long-term maintainability. Every KPI ties back to a real operational or commercial decision.

---

# 📂 Featured Patterns

| Pattern | Problem Domain | Focus | Key Techniques |
|---------|----------------|-------|----------------|
| [Account Rotation (3+3)](./account-rotation-3plus3) | CRM / Sales | Account health scoring | SWITCH classification, inactive relationships, weighted scoring, dynamic RLS |
| [Recurring-Demand Forecasting](./bunker-opportunities-overview) | Asset & service operations | Demand prediction | Calculated tables, learned baselines, multi-source coalescing |
| [Cash Utilization Tracker](./cash-utilization-tracker) | Finance | Working capital | Running totals, TREATAS, forward projection, behavioral modeling |

---

# 🔍 Pattern Overviews

---

## 1️⃣ Account Rotation (3+3) — Account Health Scoring

**The problem:**
Retail churn models assume frequent transactions. B2B relationships often don't have them — an account might order three times a year and still be perfectly healthy. Recency alone misclassifies these accounts constantly.

**The approach:**
Evaluate **engagement** and **conversion** as two sequential 90-day windows rather than one. Accounts classify as *Healthy*, *At Risk*, *Rotatable*, *Reassigned*, or *New*. The split matters operationally: an account that inquires but never orders is a sales problem; one that stops inquiring is a relationship problem. Each needs a different intervention.

**Techniques demonstrated:**
- Multi-condition classification with `SWITCH ( TRUE() )` and layered VARs
- Calculated columns for status that works in slicers and row filters
- Inactive relationships with `USERELATIONSHIP()` for multiple fact tables on one date dimension
- Weighted priority scoring combining categorical status with continuous history
- Dynamic RLS via `USERPRINCIPALNAME()` with manager hierarchy traversal

🔗 **[View pattern →](./account-rotation-3plus3)**

---

## 2️⃣ Recurring-Demand Forecasting

**The problem:**
Predicting when an asset will next need a consumable service means answering two questions at once: which assets are due, and when will they be somewhere you can serve them. Fixed rules fail immediately, because consumption rates vary enormously by asset class — a rule tuned for one class flags the rest far too early or far too late.

**The approach:**
Stop writing the rule. Derive each class's typical interval from observed service history, then measure every asset against its own class baseline. Illustrated with a maritime refueling example; the same model applies to fleet maintenance, equipment servicing, and replenishment generally.

**Techniques demonstrated:**
- Calculated tables that learn parameters from data instead of hardcoding them
- Per-class baselines via `SUMMARIZE` + `AVERAGEX` + `EARLIER`
- Gap-to-gap interval calculation with self-referencing `FILTER`
- Multi-source coalescing where feeds have partial coverage
- Tiered alerting driven by multipliers on a learned baseline

🔗 **[View pattern →](./bunker-opportunities-overview)**

---

## 3️⃣ Cash Utilization Tracker — Working Capital Visibility

**The problem:**
In any business that pays suppliers before customers pay them, cash is the binding constraint. The reporting question is deceptively simple — *how much is left, and when does it run out* — but answering it requires separating four things most systems mix together: money expected out, money actually out, money expected in, money actually in.

**The approach:**
A star schema split along the actual/expected axis, so one chart shows historical utilization and a forward projection without conditional logic fighting the filter context. Collection dates come from each customer's observed payment behavior rather than their invoice terms.

**Techniques demonstrated:**
- Cumulative aggregation with `CALCULATE` + `REMOVEFILTERS` + `KEEPFILTERS`
- `TREATAS` virtual relationships for disconnected dimensions
- Calculated tables aggregating behavioral history
- Forward projections combining locked actuals with expected movements
- Parameterized alert thresholds rather than hardcoded business rules

🔗 **[View pattern →](./cash-utilization-tracker)**

---

## 📘 Also here

**[BI Governance & Standards](./bi-governance)** — naming conventions, DAX best practices, an RLS implementation guide, onboarding checklists, and a data dictionary template for teams scaling a Power BI estate.

---

# 🎯 What I Deliver

- **Scalable data models** — star schemas, proper relationships, optimized DAX
- **Business-focused dashboards** — KPIs tied to real decisions, not vanity metrics
- **Row-level security** — enterprise-ready access control
- **Proactive alerting** — Power Automate integration for notifications
- **Clean data pipelines** — Power Query and Fabric Dataflow Gen2 transformations from messy sources
- **Documentation** — models that others can maintain and extend

---

## 🔒 Note on confidentiality

These write-ups document modeling patterns. They contain no production dataset, schema, data feed, or business rule from any employer. Table names are generic, thresholds are illustrative and parameterized, and screenshots — where included — are redacted.

---

## 📬 Contact

**Zia Malik**
📧 [zulhaq@gmail.com](mailto:zulhaq@gmail.com)
🔗 [LinkedIn](https://www.linkedin.com/in/mziamalik)

---

Thank you for visiting my portfolio!
