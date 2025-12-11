# 👋 Hi, I'm Zia — Data Strategy & Insights Analyst | Power BI Developer

Welcome to my analytics portfolio.  

I specialize in transforming raw data into clear, actionable insights using **Power BI, DAX, Power Query, and SQL**. My work focuses on solving real business problems across **Trading, Finance, Operations, and CRM** by building scalable, automated, and data-driven reporting solutions.

This repository showcases selected projects that demonstrate my technical capability, business understanding, and end-to-end BI solution design.

---

## ⚙️ Skills & Tools

<p align="left">
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=Power%20BI&logoColor=black"/>
  <img src="https://img.shields.io/badge/DAX-0A0A0A?style=for-the-badge&logo=Microsoft&logoColor=white"/>
  <img src="https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white"/>
  <img src="https://img.shields.io/badge/SQL-336791?style=for-the-badge&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white"/>
  <img src="https://img.shields.io/badge/SharePoint-0078D4?style=for-the-badge&logo=microsoft-sharepoint&logoColor=white"/>
  <img src="https://img.shields.io/badge/Power%20Automate-0066FF?style=for-the-badge&logo=power-automate&logoColor=white"/>
</p>

---

## 💡 My Approach to Analytics

I build BI solutions that balance **technical rigor with business usability**. Dashboards prioritize actionable insights, not data overload, and models are designed for long-term maintainability. Every KPI ties back to a real operational or commercial decision.

---

## 📊 Portfolio Preview

<p align="center">
  <img src="account-rotation-3plus3/Account-Rotation-Screenshot.png" width="850">
</p>
<p align="center"><i>A glimpse of my Power BI dashboards — clear, structured, and business-focused.</i></p>

---

# 📂 Featured Projects

| Project | Industry | Focus Area | Key Technologies |
|---------|----------|------------|------------------|
| [Account Rotation (3+3)](#1%EF%B8%8F⃣-account-rotation-33--customer-engagement-model) | CRM / Sales | Customer Engagement | Power BI, DAX, Time Intelligence |
| [Bunker Opportunities](#2%EF%B8%8F⃣-bunker-opportunities--maritime-demand-intelligence) | Maritime Trading | Demand Intelligence | Power BI, DAX, Dynamic Thresholds |
| [Cash Utilization](#3%EF%B8%8F⃣-cash-utilization-tracker--liquidity-forecasting) | Finance | Liquidity Management | Power BI, DAX, Running Totals |

---

# 🔍 Project Overviews

---

## 1️⃣ **Account Rotation (3+3) – Customer Engagement Model**

**Challenge:**  
Sales teams lacked a structured way to identify inactive or declining accounts and distribute trader workload based on engagement trends.

**Solution:**  
A Power BI model evaluating customer activity across 90-day enquiry and 90-day deal windows. Accounts are classified as:  
**Healthy**, **Rotatable**, **At Risk**, **Reassigned**, or **New Account** — with automated alerts and workload insights.

**Technical Highlights:**
- Complex DAX classification using SWITCH, CALCULATE, and time intelligence
- Handles 2,000+ accounts with dynamic status calculation
- Star schema with RLS enabling multi-trader, multi-office views
- Alert system for proactive account management

**Impact:**  
Reduced manual account review from **6 hours to 20 minutes weekly**, improving retention follow-up strategies.

🔗 **[View Project →](./account-rotation-3plus3)**

---

## 2️⃣ **Bunker Opportunities – Maritime Demand Intelligence**

**Challenge:**  
Traders needed a way to identify high-probability bunker leads from tens of thousands of vessel movements daily.

**Solution:**  
A maritime intelligence dashboard analyzing vessel movement, ETA forecasts, last bunker history, and STS operations.  
Includes a **High Potential Engine** using dynamic thresholds calculated from historical vessel-type behavior.

**Technical Highlights:**
- Dynamic threshold calculation using SUMMARIZE + AVERAGEX patterns
- Calculated table for vessel-type-specific bunker intervals
- Power Query processing 50,000+ vessel updates daily
- RLS for trader-specific portfolio views
- Alert system for upcoming bunker opportunities

**Impact:**  
Improved trader efficiency by **30–40%** through automated opportunity prioritization.

🔗 **[View Project →](./bunker-opportunities)**

---

## 3️⃣ **Cash Utilization Tracker – Liquidity Forecasting**

**Challenge:**  
Finance teams needed better visibility into cash allocation, supplier outflows, collection timelines, and forward exposure.

**Solution:**  
A Power BI dashboard tracking **running utilization**, **allocation limits**, **90-day projections**, and **overdue AR/AP**, with expected collection date modeling based on customer payment behavior.

**Technical Highlights:**
- Running totals with CALCULATE + REMOVEFILTERS pattern
- TREATAS for virtual relationships across disconnected tables
- Calculated table aggregating customer payment history
- Forward projection combining actuals with expected cash flows
- Finance alert system for liquidity risk management

**Impact:**  
Helped prevent **3–5 potential cash shortfalls per quarter** through proactive monitoring and alerts.

🔗 **[View Project →](./cash-utilization)**

---

# 🎯 What I Deliver

- **Scalable data models** — Star schemas, proper relationships, optimized DAX
- **Business-focused dashboards** — KPIs tied to real decisions, not vanity metrics
- **Row-Level Security** — Enterprise-ready access control
- **Proactive alerting** — Power Automate integration for notifications
- **Clean data pipelines** — Power Query transformations from messy sources
- **Documentation** — Models that others can maintain and extend

---

## 📬 Contact

**Zia Malik**  
📧 [zulhaq@gmail.com](mailto:zulhaq@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/mziamalik)

---

Thank you for visiting my portfolio!



# 📘 BI Governance & Standards

A centralized framework of **best practices, governance policies, and reusable templates** for managing enterprise Power BI deployments at scale.

---

## 🎯 Purpose

This framework solves the most common challenges BI teams face as they grow:

| Challenge | Governance Solution |
|----------|----------------------|
| Inconsistent report design across teams | **Data Modeling Standards** |
| Slow, unoptimized, or unclear DAX | **DAX Best Practices** |
| Security gaps and incorrect access | **RLS Implementation Guide** |
| Knowledge silos and slow onboarding | **Team Onboarding Guide** |
| Undocumented datasets | **Data Dictionary Template** |

---

## 📁 Repository Contents

| Document | Description | Audience |
|----------|-------------|----------|
| **[Data Modeling Standards](./data-modeling-standards.md)** | Schema design, naming conventions, relationship rules | BI Developers |
| **[DAX Best Practices](./dax-best-practices.md)** | Performance guidelines, formatting, common pitfalls | BI Developers |
| **[RLS Implementation Guide](./rls-implementation-guide.md)** | RLS strategy, testing flow, audit checklist | BI Leads, Admins |
| **[Team Onboarding Guide](./team-onboarding-guide.md)** | New analyst setup, tool configuration, standards overview | New BI Team Members |
| **[Data Dictionary Template](./data-dictionary-template.md)** | Standard format to document tables, measures, fields | All BI Developers |

---

## 🏗️ Implementation Roadmap

### Phase 1 — Foundation (Week 1–2)

- Apply naming conventions to new datasets and models  
- Adopt DAX formatting & performance patterns  
- Build data dictionaries for high-impact existing reports  

### Phase 2 — Security (Week 3–4)

- Audit all existing RLS implementations  
- Standardize dynamic RLS and role structures  
- Document workspace access matrix  

### Phase 3 — Governance (Month 2)

- Introduce code review workflow  
- Implement report certification process  
- Configure refresh monitoring and SLA alerts  

### Phase 4 — Enablement (Ongoing)

- Conduct team training on standards  
- Quarterly governance reviews  
- Continuous improvement based on user feedback  

---

## 📊 BI Maturity Assessment

| Level | Characteristics | Goal |
|-------|-----------------|------|
| **1 — Ad Hoc** | Excel-driven, no central BI strategy | Establish foundational reporting |
| **2 — Developing** | Power BI adoption growing, inconsistent outputs | Introduce governance basics |
| **3 — Defined** | Formal standards, documented datasets, RLS implemented | Scale self-service BI |
| **4 — Managed** | Automated refreshes, SLAs, auditing | Optimize enterprise BI |
| **5 — Optimized** | BI integrated into decision-making | Continuous improvement |

---

## 📈 Measuring BI Success

| Metric | What It Measures | Target |
|--------|------------------|--------|
| **Report Adoption Rate** | Active users vs total users | >70% |
| **Time to Insight** | Requirement → Dashboard delivered | <2 weeks |
| **Self-Service Ratio** | User-built vs. central BI reports | 60/40 |
| **Data Freshness SLA** | % of datasets refreshed on time | >95% |
| **Support Tickets** | BI-related issues per month | Downward trend |
| **Decision Impact** | How BI influenced decisions | Qualitative tracking |

---

## 🔄 Continuous Improvement Checklist (Quarterly)

- [ ] Update standards based on feedback  
- [ ] Audit naming conventions  
- [ ] Validate RLS configurations  
- [ ] Archive unused or duplicate reports  
- [ ] Refresh data dictionaries  
- [ ] Collect user feedback  
- [ ] Review performance (model size, DAX, refresh time)

---

## 📬 Contact

**Muhammad Zia Ul Haq**  
📧 [zulhaq@gmail.com](mailto:zulhaq@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/mziamalik)

