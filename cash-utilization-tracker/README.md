# Cash Utilization Tracker

**A Power BI pattern for working-capital visibility: running utilization, forward cash projection, and behavior-based collection forecasting.**

This is a reference implementation. The model, measures, and thresholds below are illustrative — they demonstrate a technique you can adapt, not a description of any particular deployment.

---

## The problem

In any business that pays suppliers before customers pay them, cash is the binding constraint. Payment terms differ by counterparty, collections slip, and finance teams typically discover a liquidity gap only once it's too late to do anything about it.

The reporting question is deceptively simple: *how much cash do we have left, and when does it run out?* Answering it requires separating four things most systems mix together — money expected out, money actually out, money expected in, money actually in.

---

## Data model

A star schema with one transaction dimension and four fact tables split along the actual/expected axis.

```
                    ┌──────────────┐
                    │  Sales Team  │  ◄── RLS filter point
                    │  Owner, Mgr, │
                    │ Office, Email│
                    └──────┬───────┘
                           │ 1:M
                           ▼
     ┌──────────────────────────┐      ┌────────────────────┐
     │      Transactions        │◄─────│ Capital Allocation │
     │ (central dimension:      │Office│ (Office, Amount,   │
     │  reference, status,      │      │  Start Date)       │
     │  cost, due date)         │      └────────────────────┘
     └────────────┬─────────────┘
                  │
    ┌─────────┬───┴─────┬──────────┬──────────┐
    ▼         ▼         ▼          ▼          │
┌─────────┐┌─────────┐┌──────────┐┌─────────┐ │
│Expected ││Expected ││  Actual  ││ Actual  │ │
│Outflows ││ Inflows ││ Outflows ││ Inflows │ │
└────┬────┘└────┬────┘└────┬─────┘└────┬────┘ │
     └──────────┴──────────┴───────────┴──────┘
                           │
                           ▼
                  ┌────────────────┐
                  │  Date (spine)  │
                  └────────────────┘

     ┌──────────────────────────┐
     │ Customer Payment Behavior│  ◄── Calculated table:
     │ (Avg days late, by       │      average lateness per
     │  customer)               │      customer, from history
     └──────────────────────────┘
```

| Table | Type | Purpose |
|---|---|---|
| Transactions | Dimension | Central reference linking all cash movements |
| Sales Team | Dimension | Owner hierarchy, drives RLS |
| Capital Allocation | Fact | Cash limits by office or region |
| Expected Outflows | Fact | Supplier obligations not yet paid |
| Expected Inflows | Fact | Customer invoices not yet collected |
| Actual Outflows | Fact | Payments made |
| Actual Inflows | Fact | Payments received |
| Date | Dimension | Date spine for time intelligence |
| Customer Payment Behavior | Calculated | Historical lateness per customer |

**Why split actual from expected?** Actuals drive the historical utilization line. Expectations drive the forward projection. Keeping them in separate tables means one chart can show both — solid line to today, dashed line after — without conditional logic fighting the filter context.

---

## Core measures

### Running totals

The foundation. `REMOVEFILTERS` on the date dimension plus `KEEPFILTERS` on a date comparison gives a cumulative total that respects every other filter in context:

```dax
Running Outflows =
VAR _CurrentDate = MAX ( 'Date'[Date] )
RETURN
CALCULATE (
    SUM ( 'Actual Outflows'[Amount] ),
    REMOVEFILTERS ( 'Date' ),
    KEEPFILTERS ( 'Date'[Date] <= _CurrentDate )
)
```

```dax
Running Inflows =
VAR _CurrentDate = MAX ( 'Date'[Date] )
RETURN
CALCULATE (
    SUM ( 'Actual Inflows'[Amount] ),
    REMOVEFILTERS ( 'Date' ),
    KEEPFILTERS ( 'Date'[Date] <= _CurrentDate )
)
```

The same shape applies to `Running Expected Outflows` and `Running Expected Inflows`.

### Utilization

```dax
Running Utilization =
VAR _Today = TODAY()
VAR _CurrentDate =
    IF (
        HASONEVALUE ( 'Date'[Date] ),
        MAX ( 'Date'[Date] ),
        _Today          -- card visuals have no date context
    )
VAR _Value = [Running Outflows] - [Running Inflows]
RETURN
    IF ( _CurrentDate <= _Today, _Value, BLANK() )
```

Blanking future dates is what keeps the actuals line from running flat across the forecast window.

### Allocation across a disconnected table

`Capital Allocation` is keyed on Office, not on the star schema. `TREATAS` builds the virtual relationship so the allocation respects the same Office filter the rest of the model is under:

```dax
Allocated Capital =
CALCULATE (
    SUM ( 'Capital Allocation'[Amount] ),
    KEEPFILTERS (
        TREATAS (
            VALUES ( 'Sales Team'[Office] ),
            'Capital Allocation'[Office]
        )
    )
)
```

```dax
Available Capital =
VAR _Allocated = [Allocated Capital]
VAR _Utilized  = [Running Utilization]
RETURN
    MIN ( _Allocated - _Utilized, _Allocated )
```

The `MIN` guard stops available capital exceeding the allocation when collections temporarily outpace payments.

### Overdue receivables

Iterating a filtered transaction list and netting cost against receipts per transaction:

```dax
Overdue Receivables =
VAR _Today = TODAY()
VAR _DueTransactions =
    CALCULATETABLE (
        VALUES ( 'Transactions'[Reference] ),
        'Transactions'[Invoice Due Date] < _Today,
        NOT ( 'Transactions'[Status] IN { "Completed", "Cancelled" } ),
        REMOVEFILTERS ( 'Date'[Date] )
    )
RETURN
SUMX (
    _DueTransactions,
    VAR _Ref = [Reference]
    VAR _Cost =
        CALCULATE (
            SUM ( 'Transactions'[Cost] ),
            KEEPFILTERS ( 'Transactions'[Reference] = _Ref )
        )
    VAR _Received =
        CALCULATE (
            SUM ( 'Actual Inflows'[Amount] ),
            TREATAS ( ROW ( "Reference", _Ref ), 'Actual Inflows'[Reference] ),
            KEEPFILTERS ( 'Actual Inflows'[Date] <= _Today )
        )
    RETURN MAX ( 0, _Cost - COALESCE ( _Received, 0 ) )
)
```

### Forward projection

Locks utilization as of today, then layers expected movements on top:

```dax
Expected Utilization =
VAR _Today    = TODAY()
VAR _AsOfDate = MAX ( 'Date'[Date] )

VAR _UtilizationToday =
    CALCULATE (
        [Running Utilization],
        REMOVEFILTERS ( 'Date' ),
        'Date'[Date] <= _Today
    )

VAR _ExpectedOut =
    CALCULATE (
        SUM ( 'Expected Outflows'[Amount] ),
        REMOVEFILTERS ( 'Date' ),
        'Expected Outflows'[Date] > _Today,
        'Expected Outflows'[Date] <= _AsOfDate
    )

VAR _ExpectedIn =
    CALCULATE (
        SUM ( 'Expected Inflows'[Amount] ),
        REMOVEFILTERS ( 'Date' ),
        'Expected Inflows'[Invoice Due Date] > _Today,
        'Expected Inflows'[Invoice Due Date] <= _AsOfDate
    )

RETURN
    IF (
        _AsOfDate <= _Today,
        BLANK (),
        MAX ( 0, _UtilizationToday + _ExpectedOut - _ExpectedIn )
    )
```

Cap the window by adding an end-date bound (`_Today + N`) for a fixed-horizon variant.

### Behavior-based collection dates

The interesting part. Invoice due dates are fiction — customers pay when they pay. Averaging each customer's historical lateness produces a far more realistic projection than assuming everyone pays on time:

```dax
Customer Payment Behavior =           -- calculated table
SUMMARIZE (
    FILTER (
        'Transactions',
        'Transactions'[Status] = "Completed"
            && NOT ISBLANK ( 'Transactions'[Days Late] )
    ),
    'Transactions'[Customer Key],
    "Avg Days Late",
        VAR _Avg = AVERAGE ( 'Transactions'[Days Late] )
        RETURN IF ( _Avg < 0, 0, _Avg )
)
```

```dax
Expected Collection Date =            -- calculated column on Expected Inflows
VAR _Ref = 'Expected Inflows'[Reference]
VAR _CustomerKey =
    LOOKUPVALUE ( 'Transactions'[Customer Key], 'Transactions'[Reference], _Ref )
VAR _AvgLate =
    LOOKUPVALUE (
        'Customer Payment Behavior'[Avg Days Late],
        'Customer Payment Behavior'[Customer Key], _CustomerKey
    )
RETURN
    IF (
        NOT ISBLANK ( _AvgLate ),
        'Expected Inflows'[Invoice Due Date] + INT ( _AvgLate ),
        'Expected Inflows'[Invoice Due Date]
    )
```

---

## Report layout

**KPI row** — Allocated Capital, Money Deployed (`Running Utilization`), Available Capital, Overdue Receivables.

**Utilization vs. allocation** — area chart with `Running Utilization` solid to today, `Expected Utilization` dashed beyond it, and `Allocated Capital` as a constant line ceiling. A vertical marker at `TODAY()` separates actuals from forecast.

**Three detail tables** — overdue receivables sorted by amount descending, upcoming collections with predicted payment dates, upcoming payables.

**Office slicer** — regional cash positions.

---

## Alerting

Thresholds below are **parameters, not recommendations** — set them from your own risk tolerance. Exposing them as a disconnected parameter table lets finance tune them without a model change.

```dax
Utilization Alert =
VAR _Pct = DIVIDE ( [Running Utilization], [Allocated Capital], 0 )
RETURN
SWITCH (
    TRUE(),
    _Pct >= [Critical Threshold], "Critical",
    _Pct >= [Warning Threshold],  "Warning",
    _Pct >= [Watch Threshold],    "Watch",
    "Healthy"
)
```

```dax
Days Until Capital Exhausted =
VAR _Available = [Available Capital]
VAR _DailyBurn =
    DIVIDE ( [Running Utilization], [Days Elapsed In Period], 0 )
RETURN
    IF ( _DailyBurn > 0, ROUND ( DIVIDE ( _Available, _DailyBurn, 0 ), 0 ), BLANK() )
```

A Power Automate flow querying the dataset on a schedule can push these to email or Teams when a level trips — useful for treasury teams who won't open a dashboard daily.

---

## Techniques demonstrated

- Cumulative aggregation with `CALCULATE` + `REMOVEFILTERS` + `KEEPFILTERS`
- Virtual relationships via `TREATAS` for disconnected dimensions
- Calculated tables (`SUMMARIZE`) to aggregate behavioral history
- Forward projections that combine locked actuals with expected movements
- Row-context iteration (`SUMX`) over a filtered key list
- Parameterized thresholds rather than hardcoded business rules
- Blanking strategies to keep actuals and forecasts visually distinct

---

## Stack

Power BI Desktop · DAX · Power Query (M) · star schema · scheduled refresh · optional Power Automate alerting

---

**Note:** This write-up documents a reusable modeling pattern. Table names, thresholds, and figures are illustrative. No production dataset, schema, or business rule from any employer is reproduced here.
