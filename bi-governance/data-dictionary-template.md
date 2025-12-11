# 📖 Data Dictionary Template

Use this template to document every Power BI dataset. A well-maintained data dictionary reduces support requests, speeds onboarding, and ensures consistent understanding across teams.

---

## 📋 Dataset Overview

| Field | Value |
|-------|-------|
| **Dataset Name** | [e.g., Cash Utilization Tracker] |
| **Owner** | [Name, Email] |
| **Last Updated** | [Date] |
| **Refresh Schedule** | [e.g., Daily at 6:00 AM UTC] |
| **Workspace** | [e.g., Finance - Production] |
| **Row-Level Security** | [Yes/No] |
| **Data Sources** | [e.g., ERP, SharePoint, Dataflow] |

### Business Purpose
[2-3 sentences describing what business problem this dataset solves and who uses it]

---

## 🗂️ Tables

### [Table Name 1] — [Type: Fact/Dimension]

**Purpose:** [What this table contains and why it exists]

**Source:** [Where the data comes from]

**Row Count:** [Approximate]

**Refresh:** [Incremental/Full]

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Column1` | Text | [Description] | "ABC", "XYZ" |
| `Column2` | Whole Number | [Description] | 1, 2, 3 |
| `Column3` | Date | [Description] | 2025-01-15 |
| `Column4` | Decimal | [Description] | 1234.56 |
| `Column5` | Boolean | [Description] | TRUE, FALSE |

**Business Rules:**
- [Any transformations applied]
- [Any filtering logic]
- [Any calculated columns and their logic]

---

### [Table Name 2] — [Type: Fact/Dimension]

**Purpose:** [What this table contains]

**Source:** [Where the data comes from]

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| ... | ... | ... | ... |

---

## 🔗 Relationships

| From Table | From Column | To Table | To Column | Cardinality | Direction | Active |
|------------|-------------|----------|-----------|-------------|-----------|--------|
| Deals | Customer Key | Customers | Customer Key | Many:1 | Single | Yes |
| Deals | Deal Date | Calendar | Date | Many:1 | Single | Yes |
| Deals | Invoice Date | Calendar | Date | Many:1 | Single | No |

**Notes:**
- [Explain any inactive relationships and when they're used]
- [Explain any bidirectional relationships and why]

---

## 📊 Measures

### Category: [e.g., Revenue]

| Measure | Description | Formula Summary | Format |
|---------|-------------|-----------------|--------|
| `Total Revenue` | Sum of all sales revenue | `SUM(Deals[Sales Revenue])` | Currency |
| `Revenue YTD` | Year-to-date revenue | TOTALYTD pattern | Currency |
| `Revenue vs Budget` | Variance to budget | Actual - Budget | Currency |

### Category: [e.g., Operations]

| Measure | Description | Formula Summary | Format |
|---------|-------------|-----------------|--------|
| `Deal Count` | Number of deals | `COUNTROWS(Deals)` | Number |
| `Strike Rate` | Conversion rate | Deals / Enquiries | Percentage |

### Category: [e.g., Alerts]

| Measure | Description | Formula Summary | Format |
|---------|-------------|-----------------|--------|
| `Critical Alerts` | High priority items | COUNTROWS with filter | Number |
| `Alert Summary` | Combined alert text | SWITCH pattern | Text |

---

## 🔒 Row-Level Security

### Roles Defined

| Role | Description | Filter Logic |
|------|-------------|--------------|
| Trader | Own customers only | `[Email] = USERPRINCIPALNAME()` |
| Manager | Team customers | `[Email] = UPN() OR [Manager Email] = UPN()` |
| Leadership | All data | No filter |

### Access Matrix

| Role | Customers | Deals | Financials |
|------|-----------|-------|------------|
| Trader | Own | Own | Own |
| Manager | Team | Team | Team |
| Leadership | All | All | All |

---

## 📅 Refresh Information

| Setting | Value |
|---------|-------|
| **Gateway** | [Gateway name or Direct Query] |
| **Schedule** | [e.g., Daily at 6:00 AM, 12:00 PM, 6:00 PM] |
| **Timeout** | [e.g., 2 hours] |
| **Failure Notification** | [Email addresses] |
| **Last Successful Refresh** | [Date/Time] |

### Data Latency

| Table | Source Refresh | BI Refresh | Total Latency |
|-------|----------------|------------|---------------|
| Deals | Real-time | 6:00 AM | ~6 hours max |
| Customers | Daily 5:00 AM | 6:00 AM | ~1 hour |

---

## 📝 Change Log

| Date | Author | Change Description |
|------|--------|-------------------|
| 2025-12-01 | Zia | Initial documentation |
| 2025-12-05 | Zia | Added RLS for Manager role |
| 2025-12-10 | Zia | New measure: Alert Summary |

---

## ⚠️ Known Issues & Limitations

| Issue | Impact | Workaround | Status |
|-------|--------|------------|--------|
| [Description] | [Who is affected] | [Temporary fix] | [Open/Resolved] |

---

## 📞 Support

| Type | Contact |
|------|---------|
| Data quality issues | [Data Owner] |
| Report bugs | [BI Developer] |
| Access requests | [BI Admin] |
| Business questions | [Business Owner] |

---

# Example: Cash Utilization Tracker

## 📋 Dataset Overview

| Field | Value |
|-------|-------|
| **Dataset Name** | Cash Utilization Tracker |
| **Owner** | Zia (zulhaq@gmail.com) |
| **Last Updated** | December 2025 |
| **Refresh Schedule** | Daily at 6:00 AM GST |
| **Workspace** | Finance - Production |
| **Row-Level Security** | Yes (Manager-based) |
| **Data Sources** | ERP (Dynamics BC), SharePoint |

### Business Purpose
Provides treasury and finance teams real-time visibility into cash allocation, utilization, and 90-day projections. Enables proactive liquidity management by tracking overdue receivables, upcoming payables, and available cash by office/region.

---

## 🗂️ Tables

### Deals Table — Dimension

**Purpose:** Central deal reference linking all financial transactions

**Source:** ERP System (Dynamics Business Central)

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Deal No.` | Text | Unique deal identifier | "D-2025-001" |
| `Trader` | Text | Assigned sales trader | "John Smith" |
| `Customer Key` | Text | Customer identifier | "CUST-001" |
| `Purchase Cost` | Currency | Total supplier cost | $50,000 |
| `Sales Revenue` | Currency | Total customer revenue | $55,000 |
| `Invoice Due Date` | Date | Customer payment due | 2025-01-15 |
| `Status` | Text | Deal status | "Active", "Completed" |

### Paid — Fact

**Purpose:** Actual supplier payments made

**Source:** ERP System - Supplier Payments table

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Deal No.` | Text | Related deal | "D-2025-001" |
| `Date` | Date | Payment date | 2025-01-10 |
| `Amount` | Currency | Payment amount | $50,000 |

### Receivable — Fact

**Purpose:** Expected customer collections

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Deal No.` | Text | Related deal | "D-2025-001" |
| `Invoice Due Date` | Date | When payment is due | 2025-01-15 |
| `Amount` | Currency | Expected amount | $55,000 |
| `Expected Collection Date` | Date | Predicted actual date (calculated) | 2025-01-20 |

---

## 📊 Measures

### Category: Utilization

| Measure | Description | Format |
|---------|-------------|--------|
| `Running Paid` | Cumulative supplier payments | Currency |
| `Running Receipts` | Cumulative customer receipts | Currency |
| `Running Utilization` | Net cash deployed (Paid - Receipts) | Currency |
| `Available Cash` | Allocated - Utilized | Currency |

### Category: Projections

| Measure | Description | Format |
|---------|-------------|--------|
| `Expected Utilization` | Forward projection based on payables/receivables | Currency |
| `Expected Utilization 90 Days` | Same, capped at 90-day window | Currency |

### Category: Alerts

| Measure | Description | Format |
|---------|-------------|--------|
| `Overdue Receivables` | Past-due customer amounts | Currency |
| `Cash Alert Level` | 🔴/🟠/🟡/🟢 based on utilization % | Text |

---

*Use this template as a starting point. Customize sections based on your dataset's complexity.*
