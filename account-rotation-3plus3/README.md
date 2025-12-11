# Account Rotation (3+3 Model) – Customer Engagement Analysis

A Power BI solution that evaluates customer engagement across rolling time windows to classify account health, trigger follow-up alerts, and prevent customer churn in B2B trading environments.

---

## 📊 Dashboard Preview

<p align="center">
  <img src="Account-Rotation-Screenshot.png" width="1200">
</p>
<p align="center"><i>Customer engagement dashboard showing account health, rotation timelines, trader workload, and actionable alerts.</i></p>

---

## 🏢 Business Context

In commodity trading, customer relationships require consistent engagement. Unlike retail businesses with frequent transactions, B2B trading deals are sporadic — a customer might only place orders a few times per year. This creates a challenge: **how do you identify which accounts are healthy vs. at risk of churning when transaction frequency is naturally low?**

The 3+3 rotation model solves this by:
- Evaluating **enquiry activity** (engagement signals) separately from **deal activity** (conversion signals)
- Creating structured follow-up windows that match the natural sales cycle
- Giving traders early warning before accounts go cold
- Enabling managers to balance workload and monitor team performance

---

## 🎯 What This Dashboard Delivers

| Stakeholder | Value |
|-------------|-------|
| **Traders** | Prioritized list of accounts needing follow-up, with recommended actions |
| **Managers** | Team workload visibility, at-risk account distribution, performance tracking |
| **Leadership** | Portfolio health overview, churn risk indicators, engagement trends |

---

## 🧠 The 3+3 Rotation Logic

Customers are evaluated across two sequential 90-day windows:

```
│◄────── Window 1: Enquiry ──────►│◄────── Window 2: Deal ──────►│
│         (Days 0–90)             │        (Days 91–180)          │
│                                 │                               │
│   Did customer send enquiries?  │   Did customer close deals?   │
```

### Classification Rules

| Status | Condition |
|--------|-----------|
| **Healthy** | Activity in either window (enquiry OR deal within 180 days) |
| **At Risk** | Declining engagement pattern — fewer enquiries, lower strike rate, reduced volume |
| **Rotatable** | No activity across the full 6-month period |
| **Reassigned** | Account moved to another trader |
| **New Account** | Onboarded within last 30 days — requires relationship-building focus |

---

## 📈 Key Metrics Tracked

For each customer, the model calculates:

- **Enquiry Count** — Total enquiries in evaluation period
- **Deal Count** — Closed deals in evaluation period
- **Strike Rate** — Deals ÷ Enquiries (conversion efficiency)
- **Volume Trend** — Period-over-period volume change
- **Days Since Last Enquiry** — Recency indicator
- **Days Since Last Deal** — Conversion recency
- **Days Until Rotation** — Countdown to account becoming rotatable
- **Recommended Action** — Automated follow-up guidance

---

## ⚡ Alert System

The dashboard triggers actionable alerts based on account timelines:

| Alert Type | Trigger | Recommended Action |
|------------|---------|-------------------|
| **Approaching Rotation** | 60–90 days without activity | "Follow up this week" |
| **Overdue** | 90+ days without activity | "Account overdue — take action" |
| **At Risk** | Declining KPIs despite activity | "Re-engage customer" |
| **Low Activity** | Enquiries but no deals | "Monitor — conversion issue" |
| **New Account** | Onboarded < 30 days | "Initiate relationship contact" |

---

## 🔐 Row-Level Security (RLS)

Data access is controlled by role:

| Role | Access Level |
|------|-------------|
| Trader | Own customers and vessels only |
| Manager | Entire team's accounts |
| Leadership | Full portfolio visibility |

### RLS Implementation

```dax
// Dynamic security filter applied to Customer table
[AssignedTraderEmail] = USERPRINCIPALNAME()
    || LOOKUPVALUE(
        TeamHierarchy[ManagerEmail],
        TeamHierarchy[TraderEmail], USERPRINCIPALNAME()
    ) <> BLANK()
```

---

## 🔧 Key DAX Measures

### Account Health Classification

```dax
Account Status = 
VAR DaysSinceEnquiry = DATEDIFF([LastEnquiryDate], TODAY(), DAY)
VAR DaysSinceDeal = DATEDIFF([LastDealDate], TODAY(), DAY)
VAR IsNewAccount = [AccountAge] <= 30

RETURN
SWITCH(
    TRUE(),
    IsNewAccount, "New Account",
    [IsReassigned] = TRUE(), "Reassigned",
    DaysSinceEnquiry <= 90 || DaysSinceDeal <= 90, "Healthy",
    DaysSinceEnquiry <= 180 || DaysSinceDeal <= 180, "At Risk",
    "Rotatable"
)
```

### Days Until Rotation

```dax
Days Until Rotation = 
VAR LastActivity = MAX([LastEnquiryDate], [LastDealDate])
VAR RotationDeadline = DATEADD(LastActivity, 180, DAY)
VAR DaysRemaining = DATEDIFF(TODAY(), RotationDeadline, DAY)

RETURN
IF(DaysRemaining < 0, 0, DaysRemaining)
```

### Strike Rate (Conversion Efficiency)

```dax
Strike Rate = 
DIVIDE(
    [Total Deals],
    [Total Enquiries],
    0
)
```

### Alert Priority Score

```dax
Alert Priority = 
VAR DaysRemaining = [Days Until Rotation]
VAR StatusWeight = 
    SWITCH(
        [Account Status],
        "At Risk", 100,
        "Rotatable", 80,
        "Healthy", 20,
        "New Account", 50,
        0
    )

RETURN
StatusWeight + (180 - DaysRemaining)
```

---

## 🗂 Data Model

```
┌─────────────────┐       ┌─────────────────┐
│   Customers     │───────│     Deals       │
│─────────────────│  1:M  │─────────────────│
│ CustomerID (PK) │       │ DealID (PK)     │
│ CustomerName    │       │ CustomerID (FK) │
│ AssignedTrader  │       │ DealDate        │
│ OnboardDate     │       │ Volume          │
│ Segment         │       │ Product         │
└────────┬────────┘       └─────────────────┘
         │
         │ 1:M    ┌─────────────────┐
         └────────│    Enquiries    │
                  │─────────────────│
                  │ EnquiryID (PK)  │
                  │ CustomerID (FK) │
                  │ EnquiryDate     │
                  │ Status          │
                  └─────────────────┘

┌─────────────────┐       ┌─────────────────┐
│  TraderLookup   │───────│  TeamHierarchy  │
│─────────────────│  1:1  │─────────────────│
│ TraderEmail(PK) │       │ TraderEmail     │
│ TraderName      │       │ ManagerEmail    │
│ Region          │       │ Department      │
└─────────────────┘       └─────────────────┘
```

---

## 🛠 Technical Stack

| Component | Technology |
|-----------|------------|
| Visualization | Power BI Desktop |
| Data Modeling | Star schema with role-playing dimensions |
| Calculations | DAX (time intelligence, dynamic classification, RLS) |
| Data Transformation | Power Query (M) |
| Data Source | Google Sheets (cloud-hosted anonymized data) |
| Security | Dynamic Row-Level Security |

---

## 📁 Repository Contents

```
account-rotation-3plus3/
├── Account-Rotation-3plus3.pbix    # Full Power BI report
├── Account-Rotation-Screenshot.png  # Dashboard preview
├── README.md
└── datasets/
    ├── Customers_Anonymized.csv
    ├── Deals_Anonymized.csv
    ├── Enquiries_Anonymized.csv
    └── TraderLookup.csv
```

---

## 💡 Design Decisions

1. **Why 3+3 instead of 6-month rolling?**  
   Separating enquiry and deal windows allows distinguishing between "engaged but not converting" (sales issue) vs. "not engaging at all" (relationship issue) — each requires different intervention.

2. **Why priority scoring instead of just status?**  
   Multiple accounts can be "At Risk" but urgency varies. The scoring system combines status severity with time pressure for better prioritization.

3. **Why RLS over separate reports?**  
   Single report with dynamic filtering scales better, ensures consistent metrics across roles, and simplifies maintenance.

---

## ⚠️ Disclaimer

This project is for **portfolio demonstration only**. All customer names, trader identities, volumes, dates, and business identifiers have been replaced with synthetic anonymized values. No proprietary business logic, internal systems, or confidential information is included.

---

## 📬 Contact

**Muhammad Zia Ul Haq**  
📧 [zulhaq@gmail.com](mailto:zulhaq@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/mziamalik)
