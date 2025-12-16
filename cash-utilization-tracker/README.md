# 📊 Cash Utilization Tracker

**Real-time visibility into trading cash exposure, liquidity forecasting, and collection performance.**



## 🎯 Business Problem

In commodity trading, **cash is the lifeblood of operations**. Every deal ties up working capital:

- **Supplier payments** go out before customer receipts come in
- **Credit terms** vary by customer and supplier
- **Payment delays** create unexpected liquidity gaps
- **Overdue receivables** erode available cash

Without real-time visibility, treasury and finance teams fly blind — discovering cash shortfalls only when it's too late to act.

---

## 💡 Solution

This dashboard provides a **unified cash command center** that answers critical questions:

| Question | Answer Provided |
|----------|-----------------|
| How much cash is allocated to trading? | **Allocated Cash** KPI |
| How much is currently deployed in deals? | **Money in Market** (Running Utilization) |
| What's left for new deals? | **Available Cash** |
| Who owes us money past due date? | **Overdue Receivables** table |
| When will cash run out? | **90-Day Projection** forecast line |

---

## 🗂️ Data Model

```
┌─────────────────────┐
│  SalesTraderManagers │ ◄─── RLS filter point
│  (Trader, Manager,   │
│   Office, Email)     │
└──────────┬──────────┘
           │ 1:M
           ▼
┌─────────────────────┐         ┌─────────────────┐
│    Deals Table      │◄────────│  CashAllocation │
│  (Deal No., central │  Office │  (Office, Amount│
│   deal dimension)   │         │   Start Date)   │
└──────────┬──────────┘         └─────────────────┘
           │
     ┌─────┴─────┬─────────────┬─────────────┐
     │           │             │             │
     ▼           ▼             ▼             ▼
┌─────────┐ ┌─────────┐ ┌───────────┐ ┌─────────┐
│ Payables│ │Receivable│ │   Paid    │ │ Receipts│
│(Supplier│ │(Customer │ │(Actual    │ │(Actual  │
│payments)│ │invoices) │ │ outflows) │ │ inflows)│
└────┬────┘ └────┬─────┘ └─────┬─────┘ └────┬────┘
     │           │             │             │
     └───────────┴──────┬──────┴─────────────┘
                        │
                        ▼
              ┌─────────────────┐
              │  Calendar Table │
              │  (Date spine)   │
              └─────────────────┘

Supporting Tables:
┌──────────────────────────┐
│ Customer Overdue Summary │ ◄── Calculated table
│ (Avg overdue days per    │     for collection
│  customer for forecasting│     date prediction
└──────────────────────────┘
```

### Table Purposes

| Table | Type | Purpose |
|-------|------|---------|
| **Deals Table** | Dimension | Central deal reference linking all transactions |
| **SalesTraderManagers** | Dimension | Trader hierarchy for filtering and RLS |
| **CashAllocation** | Fact | Cash limits by office/region |
| **Payables** | Fact | Expected supplier payment obligations |
| **Receivable** | Fact | Expected customer collections |
| **Paid** | Fact | Actual supplier payments made |
| **Receipts** | Fact | Actual customer payments received |
| **Calendar Table** | Dimension | Date spine for time intelligence |
| **Customer Overdue Summary** | Calculated | Historical payment behavior by customer |

---

## 📐 Key DAX Measures

### 1️⃣ Running Totals — The Foundation

**Running Paid** — Cumulative supplier payments up to each date:
```dax
Running Paid = 
VAR _CurrentDate = MAX ( 'Calendar Table'[Date] )
RETURN
CALCULATE (
    SUM ( 'Paid'[Amount] ),
    REMOVEFILTERS ( 'Calendar Table' ),
    KEEPFILTERS ( 'Calendar Table'[Date] <= _CurrentDate )
)
```

**Running Receipts** — Cumulative customer collections:
```dax
Running Receipts = 
VAR _CurrentDate = MAX ( 'Calendar Table'[Date] )
RETURN
CALCULATE(
    SUM ( 'Receipts'[Amount] ),
    FILTER (
        ALL ( 'Receipts'[Date] ),
        'Receipts'[Date] <= _CurrentDate
    )
)
```

**Running Payables** — Cumulative expected supplier obligations:
```dax
Running Payables = 
VAR _CurrentDate = MAX ( 'Calendar Table'[Date] )
RETURN
CALCULATE (
    SUM ( 'Payables'[Amount] ),
    REMOVEFILTERS ( 'Calendar Table' ),
    KEEPFILTERS ( 'Calendar Table'[Date] <= _CurrentDate )
)
```

**Running Receivable** — Cumulative expected customer invoices:
```dax
Running Receivable = 
VAR _CurrentDate = MAX ( 'Calendar Table'[Date] )
RETURN
CALCULATE (
    SUM ( 'Receivable'[Amount] ),
    REMOVEFILTERS ( 'Calendar Table' ),
    KEEPFILTERS ( 'Calendar Table'[Date] <= _CurrentDate )
)
```

---

### 2️⃣ Core Utilization Metrics

**Running Utilization** — Net cash deployed (Paid - Receipts):
```dax
Running Utilization = 
VAR _Today = TODAY()
VAR _CurrentDate =
    IF (
        HASONEVALUE ( 'Calendar Table'[Date] ),
        MAX ( 'Calendar Table'[Date] ),
        _Today        -- for card visuals or when no date context
    )
VAR _Value = [Running Paid] - [Running Receipts]
RETURN
IF (
    _CurrentDate <= _Today,
    _Value,
    BLANK()
)
```

**Allocated Cash** — Total cash available by office (respects slicer context):
```dax
Allocated Cash = 
CALCULATE(
    SUM('CashAllocation'[Amount]),
    KEEPFILTERS(
        TREATAS(
            VALUES('SalesTraderManagers'[Office]),
            'CashAllocation'[Office]
        )
    )
)
```

**Available Cash** — What's left for new deals:
```dax
Available Cash = 
VAR _Allocated = [Allocated Cash]
VAR _Utilized = [Running Utilization]
VAR _Available = _Allocated - _Utilized
RETURN
MIN ( _Available, _Allocated )
```

---

### 3️⃣ Overdue Receivables — Collections at Risk

```dax
Overdue Receivables = 
VAR _Today = TODAY()

-- deals due before today, already filtered by RLS/Manager
VAR _DueDeals =
    CALCULATETABLE(
        VALUES('Deals Table'[Deal No.]),
        'Deals Table'[Invoice Due Date] < _Today,
        NOT('Deals Table'[Status] IN {"Completed","Cancelled"}),
        REMOVEFILTERS('Calendar Table'[Date])
    )

RETURN
SUMX(
    _DueDeals,
    VAR _Deal = [Deal No.]
    VAR _Cost =
        CALCULATE(
            SUM('Deals Table'[Purchase Cost]),
            KEEPFILTERS('Deals Table'[Deal No.] = _Deal)
        )
    VAR _Paid =
        CALCULATE(
            SUM('Receipts'[Amount]),
            TREATAS(ROW("Deal No.", _Deal), 'Receipts'[Deal No.]),
            KEEPFILTERS('Receipts'[Date] <= _Today)
        )
    RETURN MAX(0, _Cost - COALESCE(_Paid,0))
)
```

---

### 4️⃣ 90-Day Projection — Forward-Looking Forecast

**Expected Utilization** — Projects future cash exposure:
```dax
Expected Utilization = 
VAR _Today    = TODAY()
VAR _AsOfDate = MAX ( 'Calendar Table'[Date] )

-- 1️⃣ Running utilization locked up to today
VAR _RU_Today =
    CALCULATE(
        [Running Utilization],
        REMOVEFILTERS ( 'Calendar Table' ),
        'Calendar Table'[Date] <= _Today
    )

-- 2️⃣ Expected supplier payments (outflows)
VAR _ExpSup_ToDate =
    CALCULATE(
        SUM ( 'Payables'[Amount] ),
        REMOVEFILTERS ( 'Calendar Table' ),
        'Payables'[Date] > _Today,
        'Payables'[Date] <= _AsOfDate
    )

-- 3️⃣ Expected customer receipts (inflows)
VAR _ExpRec_ToDate =
    CALCULATE(
        SUM ( 'Receivable'[Amount] ),
        REMOVEFILTERS ( 'Calendar Table' ),
        'Receivable'[Invoice Due Date] > _Today,
        'Receivable'[Invoice Due Date] <= _AsOfDate
    )

-- 4️⃣ Expected utilization projection
VAR _Expected = _RU_Today + _ExpSup_ToDate - _ExpRec_ToDate

-- 5️⃣ Hide past dates and prevent negative values
RETURN
IF (
    _AsOfDate <= _Today,
    BLANK (),
    MAX ( 0, _Expected )
)
```

**Expected Utilization 90 Days** — Same logic, capped at 90-day window:
```dax
Expected Utilization 90 days = 
VAR _Today    = TODAY()
VAR _AsOfDate = MAX ( 'Calendar Table'[Date] )
VAR _EndDate  = _Today + 90

-- [Same calculation as Expected Utilization...]

RETURN
IF (
    _AsOfDate < _Today || _AsOfDate > _EndDate,
    BLANK (),
    MAX ( 0, _Expected )
)
```

---

### 5️⃣ Expected Collection Date — Calculated Column

Predicts when customers will actually pay based on their historical behavior:

```dax
// Calculated column on Receivable table
Expected Collection Date = 
VAR _DealNo = 'Receivable'[Deal No.]
VAR _CustomerKey =
    LOOKUPVALUE (
        'Deals Table'[Customer Key],
        'Deals Table'[Deal No.], _DealNo
    )
VAR _AvgOverdueDays =
    LOOKUPVALUE (
        'Customer Overdue Summary'[Avg Overdue Days],
        'Customer Overdue Summary'[Customer Key], _CustomerKey
    )
VAR _InvoiceDue = 'Receivable'[Invoice Due Date]
RETURN
IF (
    NOT ISBLANK ( _AvgOverdueDays ),
    _InvoiceDue + INT ( _AvgOverdueDays ),
    _InvoiceDue        -- fallback if no avg overdue found
)
```

**Customer Overdue Summary** — Calculated table aggregating payment behavior:

```dax
Customer Overdue Summary = 
SUMMARIZE (
    FILTER (
        'Deals Table',
        'Deals Table'[Status] = "Completed"
            && NOT ISBLANK ( 'Deals Table'[Over dues] )
    ),
    'Deals Table'[Customer Key],
    "Avg Overdue Days",
        VAR _Avg = AVERAGE ( 'Deals Table'[Over dues] )
        RETURN IF ( _Avg < 0, 0, _Avg )
)
```

---

## 📊 Dashboard Components

### 1️⃣ KPI Cards (Top Row)
| Card | Measure | Purpose |
|------|---------|---------|
| **Allocated Cash** | `[Allocated Cash]` | Total cash limit |
| **Money in Market** | `[Running Utilization]` | Currently deployed |
| **Available Cash** | `[Available Cash]` | Remaining headroom |
| **Overdue Receivables** | `[Overdue Receivables]` | Collections at risk |

### 2️⃣ Utilization vs Allocation Chart
Area chart showing:
- **Running Utilization** (solid) — Historical actuals up to today
- **Expected Utilization** (dashed) — 90-day forecast
- **Allocated Cash** (horizontal line) — Cash ceiling

Visual indicator line at TODAY() separates actuals from forecast.

### 3️⃣ Overdue Receivables Table
Details of past-due invoices:
- Deal No., Trader, Vessel, Customer Name
- Invoice Due Date
- Overdue Amount
- Sorted by amount descending (biggest risks first)

### 4️⃣ Accounts Receivable Table
Upcoming customer collections:
- Invoice Due Date
- **Expected Collection Date** — Predicted actual payment date
- Amount Receivable

### 5️⃣ Accounts Payable Table
Upcoming supplier obligations:
- Deal No., Supply Trader, Supplier Name
- Invoice Due Date
- Amount Payable

### 6️⃣ Manager & Location Slicer
Filter by office/region to see regional cash positions.

---

## 🚨 Finance Team Alert System

Proactive alerts help treasury and collections teams act before issues escalate.

### Alert Categories

| Alert Level | Condition | Action Required |
|-------------|-----------|-----------------|
| 🔴 **Critical** | Available Cash < 10% of Allocated | Escalate to leadership |
| 🟠 **Warning** | Available Cash < 25% of Allocated | Review pipeline, accelerate collections |
| 🟡 **Watch** | Overdue Receivables > 15% of Receivables | Follow up with customers |
| 🔵 **Info** | Large payment due in 7 days | Ensure funds available |

### Cash Utilization Alert (Measure)

```dax
Cash Alert Level = 
VAR _Allocated = [Allocated Cash]
VAR _Available = [Available Cash]
VAR _UtilizationPct = 
    DIVIDE([Running Utilization], _Allocated, 0)

RETURN
SWITCH(
    TRUE(),
    _UtilizationPct >= 0.90, "🔴 Critical - Above 90% Utilization",
    _UtilizationPct >= 0.75, "🟠 Warning - Above 75% Utilization",
    _UtilizationPct >= 0.60, "🟡 Watch - Above 60% Utilization",
    "🟢 Healthy"
)
```

### Overdue Alert (Measure)

```dax
Overdue Alert Level = 
VAR _Overdue = [Overdue Receivables]
VAR _TotalReceivable = [Receivable Amount]
VAR _OverduePct = DIVIDE(_Overdue, _TotalReceivable, 0)

RETURN
SWITCH(
    TRUE(),
    _OverduePct >= 0.25, "🔴 Critical - 25%+ Overdue",
    _OverduePct >= 0.15, "🟠 Warning - 15%+ Overdue",
    _OverduePct >= 0.10, "🟡 Watch - 10%+ Overdue",
    "🟢 Healthy"
)
```

### Days Until Cash Exhaustion (Measure)

```dax
Days Until Cash Exhaustion = 
VAR _Available = [Available Cash]
VAR _DailyBurn = 
    DIVIDE(
        [Running Utilization],
        DATEDIFF(DATE(2025,1,1), TODAY(), DAY),
        0
    )
RETURN
IF(
    _DailyBurn > 0,
    ROUND(DIVIDE(_Available, _DailyBurn, 0), 0),
    BLANK()
)
```

### Large Payment Due Alert (Measure)

```dax
Large Payments Due 7 Days = 
VAR _Today = TODAY()
VAR _Threshold = 500000  -- $500K threshold

RETURN
CALCULATE(
    SUM('Payables'[Amount]),
    'Payables'[Date] >= _Today,
    'Payables'[Date] <= _Today + 7,
    'Payables'[Amount] >= _Threshold
)
```

### Finance Alert Summary Card

```dax
Finance Alert Summary = 
VAR _CashAlert = [Cash Alert Level]
VAR _OverdueAlert = [Overdue Alert Level]
VAR _DaysLeft = [Days Until Cash Exhaustion]
VAR _LargePayments = [Large Payments Due 7 Days]

RETURN
"Cash: " & _CashAlert & UNICHAR(10) &
"Overdue: " & _OverdueAlert & UNICHAR(10) &
"Days to Exhaustion: " & FORMAT(_DaysLeft, "#,0") & UNICHAR(10) &
"Large Payments (7d): " & FORMAT(_LargePayments, "$#,0")
```

---

## 🔄 Optional: Power Automate Integration

### Daily Cash Position Email

```
Trigger: Scheduled (daily at 7:00 AM)
Action:
  1. Query Power BI dataset for:
     - [Available Cash]
     - [Cash Alert Level]
     - [Overdue Receivables]
     - [Days Until Cash Exhaustion]
  2. If Cash Alert Level contains "Critical" or "Warning":
     - Send email to Treasury team
     - Include action items and deal details
  3. Attach PDF export of dashboard
```

### Overdue Collection Reminder

```
Trigger: Scheduled (weekly on Monday)
Action:
  1. Query Overdue Receivables > $100K
  2. Group by Customer
  3. Send personalized email to assigned Trader
  4. CC Collections team
  5. Include: Customer name, total overdue, oldest invoice date
```

### Large Payment Alert

```
Trigger: Data-driven (when Large Payments Due 7 Days > $1M)
Action:
  1. Post to Finance Teams channel
  2. Tag Treasury manager
  3. Include: Payment date, supplier, amount, deal reference
```

---

## 💡 Design Decisions

**1. Why separate Paid vs Payables, Receipts vs Receivable?**  
Actuals (Paid/Receipts) and expectations (Payables/Receivable) serve different purposes. Actuals drive the Running Utilization line; expectations drive the forecast. Keeping them separate allows accurate historical tracking while enabling forward projections.

**2. Why use Customer Overdue Summary as a calculated table?**  
Customer payment behavior varies significantly. By calculating average overdue days per customer from completed deals, the Expected Collection Date becomes realistic rather than assuming everyone pays on time.

**3. Why TREATAS for CashAllocation?**  
CashAllocation links to Office, not directly to the star schema. TREATAS creates a virtual relationship that respects the Manager/Office filter context from SalesTraderManagers.

**4. Why cap Expected Utilization at MAX(0, value)?**  
If expected collections exceed expected payments, utilization could theoretically go negative. Capping at zero prevents confusing negative values on the chart.

**5. Why show 90-day projection specifically?**  
90 days balances actionability with accuracy. Shorter windows miss medium-term liquidity events; longer windows become unreliable due to deal uncertainty.

**6. Why alert thresholds at 90%/75%/60%?**  
These thresholds provide graduated warnings. 60% gives early notice; 75% signals concern; 90% requires immediate action. Thresholds can be adjusted based on business risk tolerance.

---

## 🛠️ Technical Stack

| Component | Technology |
|-----------|------------|
| Visualization | Power BI Desktop |
| Data Modeling | DAX, Star Schema |
| Data Transformation | Power Query (M) |
| Data Sources | ERP System, SharePoint |
| Refresh | Scheduled (daily) |
| Alerts | Power Automate (optional) |

---

## 📁 Files Included

- `cash-utilization-screenshot.png` — Dashboard preview (sensitive data redacted)
- `README.md` — This documentation

*(PBIX file not included due to confidential business data)*

---

## ✅ Key Takeaways

This dashboard demonstrates:

- **Running totals with CALCULATE + REMOVEFILTERS** for cumulative aggregations
- **TREATAS** for virtual relationships across disconnected tables
- **Calculated tables (SUMMARIZE)** for aggregating historical patterns
- **Forward-looking projections** combining actuals with expected cash flows
- **Customer behavior analysis** to predict realistic collection dates
- **Multi-tier alert system** for proactive treasury management
- **Time-windowed measures** (90-day projection) for focused forecasting

The model transforms raw deal data into actionable liquidity intelligence, enabling finance teams to anticipate cash needs rather than react to shortfalls.

---

## 📬 Contact

**Zia Malik**  
📧 [zulhaq@gmail.com](mailto:zulhaq@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/mziamalik)
