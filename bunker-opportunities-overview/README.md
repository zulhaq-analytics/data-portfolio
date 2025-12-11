# Bunker Opportunities Overview – Vessel Activity & Demand Forecasting

A Power BI maritime intelligence dashboard that identifies potential bunker sales opportunities by analyzing vessel movements, ETA forecasts, historical STS (Ship-to-Ship) operations, and bunker cycle patterns. Features a custom **High Potential Engine** that dynamically scores vessels based on their likelihood to require bunkers.

---

## 📊 Dashboard Preview

<p align="center">
  <img src="possible-bunker-opportunities.png" width="1200">
</p>
<p align="center"><i>Dashboard showing vessel arrivals forecast, bunker opportunities by port, and STS operation history. Sensitive vessel and operator details redacted for confidentiality.</i></p>

---

## 🏢 Business Context

In the bunker trading industry, timing is everything. Vessels need fuel at specific points in their voyages, and traders who can anticipate these needs gain a significant competitive advantage. The challenge: **How do you predict which vessels will need bunkers and when, across thousands of vessels and dozens of ports?**

This dashboard solves that by:

- Analyzing **historical STS bunkering patterns** to establish baseline consumption cycles per vessel type
- Tracking **real-time vessel positions and ETAs** to identify upcoming arrivals at bunker ports
- Calculating **days since last bunker** to flag vessels approaching refueling needs
- Scoring opportunities using a **dynamic threshold engine** that adapts to each vessel type's typical bunker interval
- Enabling **proactive outreach** before customers even send enquiries

---

## 🎯 What This Dashboard Delivers

| Stakeholder | Value |
|-------------|-------|
| **Traders** | Prioritized list of vessels likely to need bunkers, filtered to their customer portfolio |
| **Operations** | Upcoming vessel arrivals with ETA forecasts for port planning |
| **Commercial** | Historical STS data to understand customer bunkering patterns |
| **Management** | Portfolio-wide view of opportunity pipeline by port and trader |

---

## 🗂 Data Model

```
┌─────────────────────────┐
│         IMOs            │ ◄─── Central vessel dimension (unique IMO numbers)
│─────────────────────────│
│ IMO (PK)                │
│ High Potential Flag*    │───────────────────────────────────┐
│ Vessel Type*            │                                   │
│ Trader*                 │──────────┐                        │
│ Next Arrival Date*      │          │                        │
│ Days Since Last Bunker* │          │                        ▼
│ Last Bunker Date*       │          │         ┌─────────────────────────┐
│ Last Bunker Port*       │          │         │  Dynamic_Thresholds     │
└──────────┬──────────────┘          │         │─────────────────────────│
           │                         │         │ Receiving Vessel Type   │
    ┌──────┴───────┐                 │         │ Avg Interval**          │
    │              │                 │         └─────────────────────────┘
    ▼              ▼                 │
┌─────────────────┐  ┌─────────────┐ │    ┌─────────────────────────┐
│Possible_Bunker_ │  │  STS_Table  │ │    │  Sales Trader Managers  │
│     Calls       │  │─────────────│ │    │─────────────────────────│
│─────────────────│  │Op Start Date│ │    │ Dynamics User Name      │
│ Vessel Name     │  │Op End Date  │ │    │ Sales Trader            │
│ IMO             │  │Bunker Port  │ └───►│ Role                    │
│ Current Location│  │Receiving    │      │ Manager                 │
│ Destination Port│  │ Vessel IMO  │      │ Team                    │
│ Destination ETA │  │Receiving    │      └─────────────────────────┘
│ Last Bunker Date│  │ Vessel Type │
│ Last Bunker Port│  │Days Between │
│ Operator        │  │ STS*        │
│ Vessel Type     │  │Supply Vessel│
│ DWT (mt)        │  │ Operator    │
│ Probable Status │  └─────────────┘
│ Commodity       │
│ Compliance Alert│        ┌─────────────────────────┐
└────────┬────────┘        │    Mapping_Table        │
         │                 │─────────────────────────│
         │                 │ Customer Key            │
         ▼                 │ Customer Name           │
┌─────────────────┐        │ MINT Name               │◄── Links Operators
│ Calendar Table  │        │ Trader                  │    to Customers
│─────────────────│        └─────────────────────────┘
│ Date (PK)       │
│ Year / Month    │        ┌─────────────────────────┐
│ Quarter / Week  │        │   Port Country Region   │
└─────────────────┘        │─────────────────────────│
                           │ Port                    │
* Calculated columns       │ Country                 │
** Calculated table        │ Region                  │
                           └─────────────────────────┘
```

### Relationships

| From Table | From Column | To Table | To Column | Notes |
|------------|-------------|----------|-----------|-------|
| Possible_Bunker_Calls | IMO | IMOs | IMO | Bi-directional |
| STS_Table | Receiving Vessel IMO | IMOs | IMO | |
| IMOs | Trader | Sales Trader Managers | Sales Trader | |
| Possible_Bunker_Calls | Operator | Mapping_Table | MINT Name | Customer lookup |
| Possible_Bunker_Calls | Indicated Destination Date | Calendar Table | Date | |
| Possible_Bunker_Calls | Indicated Destination Port | Port Country Region | Port | |

---

## ⭐ High Potential Engine – The Core Scoring Logic

The **High Potential Engine** is a custom scoring model that ranks vessels by their likelihood to require bunkers soon. It uses **dynamic thresholds** calculated from historical STS data rather than static rules.

### How Dynamic Thresholds Work

The system calculates average bunker intervals per vessel type from historical STS operations:

```dax
// Dynamic_Thresholds (Calculated Table)
ADDCOLUMNS(
    SUMMARIZE(
        'STS_Table',
        'STS_Table'[Receiving Vessel Type]
    ),
    "Avg Interval",
        AVERAGEX(
            FILTER(
                'STS_Table',
                'STS_Table'[Receiving Vessel Type]
                    = EARLIER('STS_Table'[Receiving Vessel Type])
                    && NOT ISBLANK('STS_Table'[Days Between STS])
            ),
            'STS_Table'[Days Between STS]
        ) * 0.75   // Reduces interval by 25% to account for incomplete STS data
)
```

### High Potential Flag (Calculated Column on IMOs)

```dax
High Potential Flag = 
VAR _Type = 'IMOs'[Vessel Type]
VAR _Next = 'IMOs'[Next Arrival Date]
VAR _Days = 'IMOs'[Days Since Last Bunker]

-- Dynamic threshold lookup from historical STS data
VAR _Threshold =
    LOOKUPVALUE(
        'Dynamic_Thresholds'[Avg Interval],
        'Dynamic_Thresholds'[Receiving Vessel Type], _Type
    )

RETURN
SWITCH(
    TRUE(),
    (
        ISBLANK(_Next)
            && _Days > _Threshold * 1.5
    )
        || (
            NOT ISBLANK(_Next)
            && _Next <= TODAY() + 14
            && (ISBLANK(_Days) || _Days > _Threshold)
        ),
    "🟢 Upcoming Bunker Need",
    "🔴 Recently Bunkered"
)
```

### Classification Logic Explained

| Flag | Condition |
|------|-----------|
| **🟢 Upcoming Bunker Need** | Vessel arriving within 14 days AND days since last bunker exceeds type-specific threshold, OR no ETA but significantly overdue (1.5x threshold) |
| **🔴 Recently Bunkered** | Vessel bunkered recently relative to its type's normal interval |

---

## 🔧 Key DAX Calculations

### Days Between STS (Calculated Column on STS_Table)

Calculates the interval between consecutive bunkering events for each vessel:

```dax
Days Between STS = 
VAR _IMO = 'STS_Table'[Receiving Vessel IMO]
VAR _Date = 'STS_Table'[Operation Start Date]
VAR _PrevDate =
    CALCULATE(
        MAX('STS_Table'[Operation Start Date]),
        FILTER(
            'STS_Table',
            'STS_Table'[Receiving Vessel IMO] = _IMO
                && 'STS_Table'[Operation Start Date] < _Date
        )
    )
RETURN
IF(NOT ISBLANK(_PrevDate),
    DATEDIFF(_PrevDate, _Date, DAY)
)
```

### Days Since Last Bunker (Calculated Column on IMOs)

```dax
Days Since Last Bunker = 
IF (
    NOT ISBLANK ( [Last Bunker Date] ),
    DATEDIFF ( [Last Bunker Date], TODAY(), DAY )
)
```

### Last Bunker Date (Calculated Column on IMOs)

Combines data from both PBC (Possible Bunker Calls) and STS sources, taking the most recent:

```dax
Last Bunker Date = 
VAR _DatePBC = IMOs[Last Bunker Date PBC]
VAR _DateSTS = IMOs[Last Bunker Date STS]
RETURN
    IF (
        _DatePBC > _DateSTS,
        _DatePBC,
        _DateSTS
    )
```

### Next Arrival Date (Calculated Column on IMOs)

```dax
Next Arrival Date = 
VAR _IMO = 'IMOs'[IMO]
RETURN
CALCULATE (
    MIN ( 'Possible_Bunker_Calls'[Indicated Destination Date] ),
    FILTER (
        'Possible_Bunker_Calls',
        'Possible_Bunker_Calls'[IMO] = _IMO
            && 'Possible_Bunker_Calls'[Indicated Destination Date] >= TODAY()
    )
)
```

### Trader Assignment (Calculated Column on IMOs)

Links vessels to traders via operator-customer mapping:

```dax
Trader = 
VAR _OperatorPBC =
    CALCULATE (
        MAX ( 'Possible_Bunker_Calls'[Operator] ),
        FILTER ( 'Possible_Bunker_Calls', 'Possible_Bunker_Calls'[IMO] = 'IMOs'[IMO] )
    )
VAR _TraderPBC =
    LOOKUPVALUE (
        'Mapping_Table'[Trader],
        'Mapping_Table'[MINT Name], _OperatorPBC
    )

VAR _OperatorSTS =
    CALCULATE (
        MAX ( 'STS_Table'[Receiving Vessel Operator] ),
        FILTER ( 'STS_Table', 'STS_Table'[Receiving Vessel IMO] = 'IMOs'[IMO] )
    )
VAR _TraderSTS =
    LOOKUPVALUE (
        'Mapping_Table'[Trader],
        'Mapping_Table'[MINT Name], _OperatorSTS
    )

VAR _FinalTrader =
    COALESCE ( _TraderPBC, _TraderSTS )

RETURN
IF ( ISBLANK ( _FinalTrader ), "Unassigned", _FinalTrader )
```

### Cumulative Vessel Arrivals (Measure)

For the arrivals trend chart:

```dax
Cumulative Vessel Arrivals = 
VAR _maxDate = MAX ( 'Calendar Table'[Date] )
RETURN
CALCULATE (
    [Vessel Arrivals],
    FILTER (
        ALLSELECTED ( 'Calendar Table'[Date] ),
        'Calendar Table'[Date] <= _maxDate
    )
)
```

---

## 📊 Dashboard Components

### 1️⃣ KPI Cards
- **🟢 Potential Bunker Opportunities** — Count of vessels flagged as "Upcoming Bunker Need"
- **🔴 Recently Bunkered** — Count of vessels that bunkered recently
- **🚨 Active Alerts** — Count of vessels requiring trader attention (High/Medium/Low priority)

### 2️⃣ Port Summary Matrix
Breakdown by destination port showing:
- Potential opportunities count
- Recently bunkered count
- Toggle between Port view and Trader view

### 3️⃣ Upcoming Vessel Arrivals Chart
Area chart showing cumulative vessel arrivals over the next 30 days, helping operations plan for upcoming demand.

### 4️⃣ Upcoming Bunker Opportunities Table
Detailed vessel list including:
- Trader, IMO, Vessel Name, Operator
- Current Location → Destination Port
- Destination ETA
- Last Bunker Date & Port
- Status (Ballast/Laden)
- Days Since Last Bunker
- Compliance Alert
- **High Potential Flag** (🟢/🔴)
- **Alert Priority** (🔴/🟠/🟡) — with conditional formatting

### 5️⃣ Possible Bunker Locations
Predicted bunkering ports based on vessel routes:
- Bunker EDA (Estimated Date of Arrival)
- Vessel Type, DWT
- Intelligence Confidence level
- Commodity being carried

### 6️⃣ Historic STS Operations
Historical bunkering activity showing:
- Bunker Date & Port
- Vessel Name & IMO
- Operator
- Supply Vessel details

### 7️⃣ Trader Alert Summary (Manager View)
Matrix showing alert distribution across traders:
- High/Medium/Low priority counts per trader
- Unassigned vessels requiring attention
- Helps managers balance workload and ensure coverage

---

## 🔒 Row-Level Security (RLS)

RLS ensures data confidentiality and personalized insights aligned with real-world operational access levels.

### Access Matrix

| Role | Customers | Vessels | Opportunities | Alerts |
|------|-----------|---------|---------------|--------|
| **Trader** | Own customers only | Vessels linked to own customers + Unassigned | Own portfolio only | Receives personal alerts |
| **Manager** | All team customers | All team vessels + Unassigned | Team-wide view | Team alerts summary |
| **Leadership** | All customers | All vessels | Full pipeline | All alerts |

### RLS Implementation

**Role: Trader**
```dax
// Filter on Sales Trader Managers table
[Email] = USERPRINCIPALNAME()
```

**Role: Manager**
```dax
// Filter on Sales Trader Managers table - sees own + team members
[Email] = USERPRINCIPALNAME()
    || [Manager Email] = USERPRINCIPALNAME()
```

**Role: Leadership**
```dax
// No filter applied - full access
// Alternatively, filter by role:
[Role] = "Leadership"
    || LOOKUPVALUE(
        'Sales Trader Managers'[Role],
        'Sales Trader Managers'[Email], USERPRINCIPALNAME()
    ) = "Leadership"
```

### Vessel Filtering Logic

Since vessels connect to traders via the `Trader` calculated column on IMOs (which maps operators to customers to traders), RLS automatically flows through the model:

```
Sales Trader Managers (RLS Filter)
        │
        ▼
      IMOs[Trader]
        │
        ▼
   Possible_Bunker_Calls
        │
        ▼
   All related visuals
```

Unassigned vessels (where `Trader = "Unassigned"`) are visible to all roles to ensure no opportunities are missed.

---

## 🚨 Alert System – Proactive Trader Notifications

The dashboard includes an alerting mechanism that flags vessels requiring immediate trader attention, enabling proactive outreach before customers send enquiries.

### Alert Categories

| Alert Level | Trigger Condition | Action Required |
|-------------|-------------------|-----------------|
| 🔴 **High Priority** | Vessel arriving within 7 days AND days since last bunker > 1.5x threshold | Contact customer immediately |
| 🟠 **Medium Priority** | Vessel arriving within 14 days AND days since last bunker > threshold | Prepare follow-up, monitor |
| 🟡 **Low Priority** | Vessel arriving within 30 days AND approaching bunker threshold | Optional follow-up |

### Alert Priority (Calculated Column on IMOs)

```dax
Alert Priority = 
VAR _Days = [Days Since Last Bunker]
VAR _ArrivalDays = DATEDIFF(TODAY(), [Next Arrival Date], DAY)
VAR _Type = [Vessel Type]
VAR _Threshold = 
    LOOKUPVALUE(
        'Dynamic_Thresholds'[Avg Interval],
        'Dynamic_Thresholds'[Receiving Vessel Type], _Type
    )

RETURN
SWITCH(
    TRUE(),
    -- High Priority: Arriving soon + significantly overdue
    _ArrivalDays <= 7 && _Days > _Threshold * 1.5, "🔴 High Priority",
    
    -- Medium Priority: Arriving within 2 weeks + overdue
    _ArrivalDays <= 14 && _Days > _Threshold, "🟠 Medium Priority",
    
    -- Low Priority: Arriving within month + approaching threshold
    _ArrivalDays <= 30 && _Days > _Threshold * 0.8, "🟡 Low Priority",
    
    -- No alert needed
    BLANK()
)
```

### Alert Triggers

Alerts are generated when:

- A vessel is **approaching its typical bunker interval** (based on vessel type)
- ETA to a bunker port is within **7–30 days**
- The vessel has **no recent bunker history** in either PBC or STS data
- Recent **STS activity** suggests the vessel is active and consuming fuel
- A vessel shows **high distance traveled since last bunker**
- **Compliance alerts** are flagged (sanctions, documentation issues)

### Recommended Action (Calculated Column on IMOs)

```dax
Recommended Action = 
VAR _AlertLevel = [Alert Priority]
VAR _Status = [High Potential Flag]
VAR _Compliance = 
    CALCULATE(
        MAX('Possible_Bunker_Calls'[Compliance Alert]),
        FILTER('Possible_Bunker_Calls', 'Possible_Bunker_Calls'[IMO] = [IMO])
    )

RETURN
SWITCH(
    TRUE(),
    NOT ISBLANK(_Compliance), "⚠️ Review compliance before contact",
    _AlertLevel = "🔴 High Priority", "📞 Contact customer immediately",
    _AlertLevel = "🟠 Medium Priority", "📧 Send follow-up email",
    _AlertLevel = "🟡 Low Priority", "👁️ Monitor - add to watchlist",
    _Status = "🟢 Upcoming Bunker Need", "📋 Prepare quotation",
    "No action required"
)
```

### Trader Alert Summary (Measure)

For KPI cards showing each trader's alert counts:

```dax
High Priority Alerts = 
COUNTROWS(
    FILTER(
        'IMOs',
        [Alert Priority] = "🔴 High Priority"
    )
)

Total Active Alerts = 
COUNTROWS(
    FILTER(
        'IMOs',
        NOT ISBLANK([Alert Priority])
    )
)
```

### Alert Distribution by Trader (Visual)

A matrix showing alert counts per trader helps managers balance workload:

| Trader | 🔴 High | 🟠 Medium | 🟡 Low | Total |
|--------|---------|-----------|--------|-------|
| Trader A | 3 | 7 | 12 | 22 |
| Trader B | 1 | 4 | 8 | 13 |
| Unassigned | 5 | 11 | 24 | 40 |

---

## 🛠 Technical Stack

| Component | Technology |
|-----------|------------|
| Visualization | Power BI Desktop |
| Data Sources | S&P Global MINT data, SharePoint Excel files, Power Platform Dataflows |
| Calculations | DAX (calculated tables, calculated columns, measures) |
| Data Transformation | Power Query (M) |
| Date Intelligence | Custom Calendar Table |
| Security | Row-Level Security |

---

## 💡 Design Decisions

**1. Why Dynamic Thresholds instead of static rules?**  
Different vessel types have vastly different consumption patterns. A VLCC burns fuel at a different rate than an LPG carrier. Using historical STS data to calculate type-specific intervals is more accurate than one-size-fits-all rules.

**2. Why 0.75 multiplier on average intervals?**  
STS data doesn't capture all bunkering events (some occur at ports rather than ship-to-ship). Reducing the threshold by 25% creates a more conservative estimate that catches opportunities earlier.

**3. Why combine PBC and STS data for Last Bunker Date?**  
Different data sources capture different bunkering events. Taking the most recent date from either source gives the most complete picture.

**4. Why bi-directional filtering on IMOs ↔ Possible_Bunker_Calls?**  
Enables slicers on vessel details to filter the arrivals chart and vice versa, creating a cohesive interactive experience.

**5. Why "Unassigned" as default trader?**  
Ensures no vessel falls through the cracks. Unassigned vessels represent potential new business that any trader can pursue.

**6. Why three-tier alert priority (High/Medium/Low)?**  
Not all opportunities are equally urgent. A vessel arriving tomorrow that's overdue for bunkers needs immediate action, while one arriving in 3 weeks can be added to a watchlist. The tiered system helps traders prioritize their daily workflow.

**7. Why alert thresholds use multipliers (1.5x, 1.0x, 0.8x)?**  
These multipliers create meaningful differentiation:
- 1.5x threshold = significantly overdue, high urgency
- 1.0x threshold = at expected interval, moderate urgency  
- 0.8x threshold = approaching interval, early warning

---

## 🔄 Optional: Power Automate Integration

For proactive notifications beyond the dashboard, the alert system can be extended with Power Automate:

### Daily Alert Email to Traders
```
Trigger: Scheduled (daily at 8:00 AM)
Action: 
  1. Query Power BI dataset for High Priority alerts per trader
  2. Send personalized email to each trader with their alert list
  3. Include vessel name, ETA, days since last bunker, recommended action
```

### Teams Notification for High Priority
```
Trigger: Data-driven alert when High Priority count > threshold
Action:
  1. Post to Trading Team channel
  2. Include vessel details and customer contact info
  3. Tag assigned trader for immediate attention
```

This creates a proactive workflow where traders receive alerts without needing to open the dashboard.

---

## 📁 Repository Contents

```
bunker-opportunities-overview/
├── possible-bunker-opportunities.png   # Dashboard preview (redacted)
├── README.md
└── (No .pbix or datasets included due to data sensitivity and size)
```

---

## ⚠️ Disclaimer

This project is for **portfolio demonstration only**. All vessel names, IMO numbers, operator names, and sensitive details in the screenshot have been redacted or blurred. The actual datasets are not included due to:
- Confidentiality of maritime intelligence data
- Large dataset size making dummy data generation impractical
- Proprietary S&P Global MINT data licensing

The DAX code and data model documentation demonstrate the technical approach and business logic.

---

## 📬 Contact

**Muhammad Zia Ul Haq**  
📧 [zulhaq@gmail.com](mailto:zulhaq@gmail.com)  
🔗 [LinkedIn](https://www.linkedin.com/in/mziamalik)
