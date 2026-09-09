# Recurring-Demand Forecasting

**A Power BI pattern for predicting when an asset will next need a consumable service, using intervals learned from its own history rather than fixed rules.**

Illustrated with a maritime refueling example. The same model applies to any recurring-service domain — fleet maintenance, equipment servicing, consumable replenishment — where assets consume at different rates and a single global rule misfires.

This is a reference implementation. Table names, thresholds, and multipliers are illustrative.

---

## The problem

Predicting demand for a recurring service means answering two questions at once: *which assets are due*, and *when will they be somewhere we can serve them*.

Fixed rules fail immediately, because consumption rates vary enormously by asset class. A rule tuned for one class flags the others far too early or far too late — and a forecast that cries wolf gets ignored within a week.

The fix is to stop writing the rule. Derive each class's typical interval from observed service history, then measure every asset against its own class baseline.

---

## Data model

```
      ┌──────────────────────────┐
      │         Assets           │  ◄── central dimension
      │──────────────────────────│
      │ Asset ID (PK)            │
      │ Asset Class *            │
      │ Owner *                  │──────────┐
      │ Next Arrival Date *      │          │
      │ Days Since Last Service *│          ▼
      │ Last Service Date *      │   ┌──────────────────────┐
      │ Demand Flag *            │   │  Interval Baselines  │ **
      └───────────┬──────────────┘   │──────────────────────│
                  │                  │ Asset Class          │
        ┌─────────┴─────────┐        │ Avg Interval         │
        ▼                   ▼        └──────────────────────┘
┌─────────────────┐  ┌──────────────┐
│Scheduled        │  │Service       │   ┌──────────────────────┐
│  Arrivals       │  │  History     │   │      Sales Team      │
│─────────────────│  │──────────────│   │──────────────────────│
│ Asset ID        │  │ Asset ID     │   │ User Name (PK)       │
│ Asset Name      │  │ Start Date   │   │ Owner / Role         │
│ Current Location│  │ End Date     │   │ Manager / Team/Email │
│ Destination     │  │ Location     │   └──────────────────────┘
│ Destination ETA │  │ Asset Class  │
│ Last Service Dt │  │ Interval *   │   ┌──────────────────────┐
│ Operator        │  │ Provider     │   │  Operator Mapping    │
│ Capacity        │  └──────────────┘   │──────────────────────│
│ Asset Class     │                     │ Account Key          │
└────────┬────────┘                     │ Account Name         │
         │                              │ Source System Name   │
         ▼                              │ Owner                │
┌─────────────────┐                     └──────────────────────┘
│      Date       │
│─────────────────│                     ┌──────────────────────┐
│ Date (PK)       │                     │  Location Reference  │
│ Year / Month    │                     │ Location/Country/Rgn │
└─────────────────┘                     └──────────────────────┘

*  calculated columns      ** calculated table
```

`Assets` ↔ `Scheduled Arrivals` uses bi-directional filtering so asset-level slicers drive the arrivals chart and vice versa.

---

## The scoring engine

### Learned baselines

A calculated table that derives the average service interval per asset class from history:

```dax
Interval Baselines =
ADDCOLUMNS (
    SUMMARIZE ( 'Service History', 'Service History'[Asset Class] ),
    "Avg Interval",
        AVERAGEX (
            FILTER (
                'Service History',
                'Service History'[Asset Class] = EARLIER ( 'Service History'[Asset Class] )
                    && NOT ISBLANK ( 'Service History'[Interval] )
            ),
            'Service History'[Interval]
        ) * [Coverage Adjustment]        -- parameter, see note below
)
```

**On the coverage adjustment.** Service history rarely captures every event — some servicing happens through channels the dataset doesn't see. That inflates the observed average interval. Multiplying by a factor below 1 pulls the baseline back to a conservative estimate.

Set this from your own data coverage. Measure it if you can: compare a sample of assets against a known-complete source and use the observed capture rate. Guessing produces a forecast that is confidently wrong.

### Interval calculation

Days between consecutive service events per asset:

```dax
Interval =                              -- calculated column on Service History
VAR _Asset = 'Service History'[Asset ID]
VAR _Date  = 'Service History'[Start Date]
VAR _Previous =
    CALCULATE (
        MAX ( 'Service History'[Start Date] ),
        FILTER (
            'Service History',
            'Service History'[Asset ID] = _Asset
                && 'Service History'[Start Date] < _Date
        )
    )
RETURN
    IF ( NOT ISBLANK ( _Previous ), DATEDIFF ( _Previous, _Date, DAY ) )
```

### Demand flag

Every asset measured against its own class baseline:

```dax
Demand Flag =                           -- calculated column on Assets
VAR _Class = 'Assets'[Asset Class]
VAR _Next  = 'Assets'[Next Arrival Date]
VAR _Days  = 'Assets'[Days Since Last Service]

VAR _Threshold =
    LOOKUPVALUE (
        'Interval Baselines'[Avg Interval],
        'Interval Baselines'[Asset Class], _Class
    )

VAR _ArrivalWindow  = [Arrival Window Days]      -- parameter
VAR _OverdueFactor  = [Overdue Factor]           -- parameter

RETURN
SWITCH (
    TRUE(),
    -- no scheduled arrival, but well past its normal interval
    ( ISBLANK ( _Next ) && _Days > _Threshold * _OverdueFactor )
        -- or arriving soon and already at interval
        || ( NOT ISBLANK ( _Next )
             && _Next <= TODAY() + _ArrivalWindow
             && ( ISBLANK ( _Days ) || _Days > _Threshold ) ),
    "Upcoming Need",
    "Recently Serviced"
)
```

### Supporting columns

```dax
Days Since Last Service =
IF (
    NOT ISBLANK ( [Last Service Date] ),
    DATEDIFF ( [Last Service Date], TODAY(), DAY )
)
```

```dax
Last Service Date =                     -- most recent across two sources
VAR _FromArrivals = Assets[Last Service Date - Arrivals]
VAR _FromHistory  = Assets[Last Service Date - History]
RETURN IF ( _FromArrivals > _FromHistory, _FromArrivals, _FromHistory )
```

Different sources capture different events. Taking the later of the two is the only way to avoid flagging an asset that was serviced through the channel your primary feed doesn't cover.

```dax
Next Arrival Date =
VAR _Asset = 'Assets'[Asset ID]
RETURN
CALCULATE (
    MIN ( 'Scheduled Arrivals'[Destination ETA] ),
    FILTER (
        'Scheduled Arrivals',
        'Scheduled Arrivals'[Asset ID] = _Asset
            && 'Scheduled Arrivals'[Destination ETA] >= TODAY()
    )
)
```

### Owner assignment

Assets link to owners indirectly — through operator, then account, then owner — and the operator may appear in either source:

```dax
Owner =
VAR _OperatorArrivals =
    CALCULATE (
        MAX ( 'Scheduled Arrivals'[Operator] ),
        FILTER ( 'Scheduled Arrivals', 'Scheduled Arrivals'[Asset ID] = 'Assets'[Asset ID] )
    )
VAR _OwnerArrivals =
    LOOKUPVALUE ( 'Operator Mapping'[Owner], 'Operator Mapping'[Source System Name], _OperatorArrivals )

VAR _OperatorHistory =
    CALCULATE (
        MAX ( 'Service History'[Provider] ),
        FILTER ( 'Service History', 'Service History'[Asset ID] = 'Assets'[Asset ID] )
    )
VAR _OwnerHistory =
    LOOKUPVALUE ( 'Operator Mapping'[Owner], 'Operator Mapping'[Source System Name], _OperatorHistory )

RETURN COALESCE ( _OwnerArrivals, _OwnerHistory, "Unassigned" )
```

`"Unassigned"` is deliberate, not a fallback for missing data. Unassigned assets are visible to every role — they represent unclaimed opportunity, and hiding them behind RLS means nobody pursues them.

### Cumulative arrivals

```dax
Cumulative Arrivals =
VAR _MaxDate = MAX ( 'Date'[Date] )
RETURN
CALCULATE (
    [Arrivals],
    FILTER ( ALLSELECTED ( 'Date'[Date] ), 'Date'[Date] <= _MaxDate )
)
```

---

## Alert priority

Three tiers, each combining arrival proximity with overdue severity:

```dax
Alert Priority =
VAR _Days        = [Days Since Last Service]
VAR _ArrivalDays = DATEDIFF ( TODAY(), [Next Arrival Date], DAY )
VAR _Threshold   =
    LOOKUPVALUE (
        'Interval Baselines'[Avg Interval],
        'Interval Baselines'[Asset Class], [Asset Class]
    )
RETURN
SWITCH (
    TRUE(),
    _ArrivalDays <= 7  && _Days > _Threshold * 1.5, "High",
    _ArrivalDays <= 14 && _Days > _Threshold,       "Medium",
    _ArrivalDays <= 30 && _Days > _Threshold * 0.8, "Low",
    BLANK()
)
```

The multipliers create meaningful separation rather than an arbitrary cutoff: above 1.5× is significantly overdue, 1.0× is at interval, 0.8× is an early warning. Expose them as parameters so the business can retune without a model change.

```dax
Recommended Action =
VAR _Priority = [Alert Priority]
VAR _Flag     = [Demand Flag]
RETURN
SWITCH (
    TRUE(),
    NOT ISBLANK ( [Compliance Flag] ), "Review compliance before contact",
    _Priority = "High",                "Contact immediately",
    _Priority = "Medium",              "Send follow-up",
    _Priority = "Low",                 "Add to watchlist",
    _Flag = "Upcoming Need",           "Prepare quotation",
    "No action required"
)
```

---

## Report layout

**KPI row** — upcoming needs, recently serviced, active alerts.

**Location matrix** — opportunity counts by destination, toggling between location view and owner view.

**Arrivals trend** — cumulative arrivals over the next 30 days, for capacity planning.

**Opportunity table** — owner, asset, operator, current location → destination, ETA, last service date and location, days since last service, demand flag, alert priority with conditional formatting.

**Service history** — past service events for pattern context.

**Owner alert matrix** — alert distribution across the team, including unassigned, so managers can balance workload.

---

## Row-level security

| Role | Accounts | Assets | Pipeline |
|---|---|---|---|
| Owner | Own only | Own + unassigned | Own only |
| Manager | Team | Team + unassigned | Team-wide |
| Leadership | All | All | Full |

```dax
-- Role: Owner
[Email] = USERPRINCIPALNAME()

-- Role: Manager
[Email] = USERPRINCIPALNAME() || [Manager Email] = USERPRINCIPALNAME()

-- Role: Leadership
LOOKUPVALUE ( 'Sales Team'[Role], 'Sales Team'[Email], USERPRINCIPALNAME() ) = "Leadership"
```

The filter reaches assets through the `Owner` calculated column, and from there to every visual.

---

## Techniques demonstrated

- Calculated tables that learn parameters from data instead of hardcoding them
- Per-class baselines via `SUMMARIZE` + `AVERAGEX` + `EARLIER`
- Gap-to-gap interval calculation with self-referencing `FILTER`
- Multi-source coalescing where feeds have partial coverage
- Indirect relationship traversal (asset → operator → account → owner)
- Tiered alerting driven by multipliers on a learned baseline
- Bi-directional filtering for cohesive cross-visual interaction
- Deliberate handling of unassigned records rather than hiding them

---

## Stack

Power BI Desktop · DAX (calculated tables, columns, measures) · Power Query (M) · star schema · dynamic RLS · optional Power Automate alerting

---

**Note:** This write-up documents a reusable modeling pattern, illustrated with a maritime example. Table names, thresholds, and multipliers are illustrative. No production dataset, schema, data feed, or business rule from any employer is reproduced here.
