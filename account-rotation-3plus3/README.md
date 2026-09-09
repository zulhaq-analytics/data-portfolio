# Account Rotation (3+3 Model)

**A Power BI pattern for scoring account health in low-frequency B2B relationships, where transaction counts are too sparse for conventional churn metrics.**

This is a reference implementation. Table names, thresholds, and scoring weights are illustrative — adapt them to your own sales cycle.

---

## The problem

Retail churn models assume frequent transactions. B2B relationships often don't have them: an account might order three times a year and still be perfectly healthy. Recency alone misclassifies these accounts constantly.

The 3+3 model handles this by evaluating **engagement** and **conversion** as two sequential windows rather than one. That distinction matters operationally:

- An account that inquires but never orders is a **sales** problem — pricing, competitiveness, responsiveness.
- An account that stops inquiring at all is a **relationship** problem — needs outreach, or reassignment.

Collapsing both into "days since last order" tells you an account is cold, but not why, and the two need different interventions.

---

## The logic

Two sequential 90-day windows:

```
│◄──── Window 1: Engagement ────►│◄──── Window 2: Conversion ────►│
│         (Days 0–90)            │         (Days 91–180)          │
│                                │                                │
│   Did the account inquire?     │   Did the inquiry convert?     │
│                                │                                │
│  Reference: last order date    │  Deadline: inquiry + 90 days   │
│  (or date account was added)   │  (or reference + 90 days)      │
```

| Status | Condition |
|---|---|
| Healthy | More than 30 days until deadline |
| At Risk | 30 days or fewer remaining, not yet past |
| Rotatable | Deadline passed |
| Reassigned | Within grace period after an ownership change |
| New | Added within the grace window, never reassigned |

The grace period exists so a newly reassigned account isn't immediately flagged against the previous owner's inactivity.

---

## Data model

```
        ┌──────────────────────┐
        │      Accounts        │  ◄── central dimension
        │──────────────────────│
        │ Account Key (PK)     │
        │ Account Name         │
        │ Owner                │──────────────┐
        │ Region               │              │
        │ Date Added           │──┐           ▼
        │ Status *             │  │   ┌────────────────┐
        │ Engagement Level *   │  │   │     Owners     │
        │ Rotation Urgency *   │  │   │────────────────│
        └──────────┬───────────┘  │   │ User Name (PK) │
                   │              │   │ Owner          │
          ┌────────┴────────┐     │   │ Role           │
          ▼                 ▼     │   │ Manager        │
    ┌──────────┐     ┌───────────┐│   │ Team / Email   │
    │  Orders  │     │ Inquiries ││   └────────────────┘
    │──────────│     │───────────││
    │ Order No │     │ Inquiry No││   ┌────────────────┐
    │ Order Dt │     │ Inquiry Dt││   │      Date      │
    │ Volume   │     │ Status    │└──►│────────────────│
    │ Acct Key │     │ Acct Key  │    │ Date (PK)      │
    └──────────┘     └───────────┘    │ YearMonth      │
                                      └────────────────┘
        ┌──────────────────────┐            ▲
        │  Ownership Changes   │            │
        │──────────────────────│            │
        │ Account Key          │────────────┘
        │ Change Date          │
        └──────────────────────┘

* calculated columns
```

Both `Orders` and `Inquiries` connect to `Date` via **inactive** relationships — two fact tables can't both hold an active path to the same dimension. Measures activate the one they need with `USERELATIONSHIP()`.

---

## Core logic

### Status classification

The whole model rests on this calculated column:

```dax
Status =
VAR _DateAdded     = [Date Added]
VAR _LastInquiry   = CALCULATE ( MAX ( Inquiries[Inquiry Date] ) )
VAR _LastOrder     = CALCULATE ( MAX ( Orders[Order Date] ) )
VAR _LastReassign  = CALCULATE ( MAX ( 'Ownership Changes'[Change Date] ) )

VAR _WindowDays  = 90
VAR _AtRiskDays  = 30

VAR _InGracePeriod =
    NOT ISBLANK ( _LastReassign )
        && DATEDIFF ( _LastReassign, TODAY(), DAY ) < _WindowDays

VAR _IsNew =
    DATEDIFF ( _DateAdded, TODAY(), DAY ) < _WindowDays
        && ISBLANK ( _LastReassign )

-- priority: recent reassignment > last order > date added
VAR _ReferenceDate =
    IF (
        _InGracePeriod,
        _LastReassign,
        IF ( ISBLANK ( _LastOrder ), _DateAdded, _LastOrder )
    )

VAR _HasValidInquiry =
    NOT ISBLANK ( _LastInquiry )
        && _LastInquiry > _ReferenceDate
        && DATEDIFF ( _ReferenceDate, _LastInquiry, DAY ) <= _WindowDays

-- if a valid inquiry exists, the clock runs from the inquiry
VAR _Deadline =
    IF (
        _HasValidInquiry,
        _LastInquiry + _WindowDays,
        _ReferenceDate + _WindowDays
    )

VAR _DaysRemaining = DATEDIFF ( TODAY(), _Deadline, DAY )

RETURN
SWITCH (
    TRUE(),
    _InGracePeriod,                "Reassigned",
    _IsNew,                        "New",
    _DaysRemaining <= 0,           "Rotatable",
    _DaysRemaining <= _AtRiskDays, "At Risk",
    "Healthy"
)
```

**Why a calculated column rather than a measure?** Status has to work in slicers, row filters, and conditional formatting. Those need row context that persists — a measure re-evaluates per visual and can't be sliced on.

`Days Until Rotation` reuses the identical deadline logic and returns `_DaysRemaining` directly. In production, factor the shared portion into a calculation group or a helper column to avoid maintaining the same VAR block twice.

### Engagement level

Activity intensity over a rolling 180 days, independent of the rotation clock:

```dax
Engagement Level =
VAR _Key      = 'Accounts'[Account Key]
VAR _Lookback = 180

VAR _Inquiries =
    CALCULATE (
        COUNTROWS ( Inquiries ),
        ALL ( Inquiries ),
        Inquiries[Account Key] = _Key,
        Inquiries[Inquiry Date] >= TODAY() - _Lookback
    )

VAR _Converted =
    CALCULATE (
        COUNTROWS ( Inquiries ),
        ALL ( Inquiries ),
        Inquiries[Account Key] = _Key,
        Inquiries[Status] = "Converted",
        Inquiries[Inquiry Date] >= TODAY() - _Lookback
    )

VAR _Base =
    SWITCH (
        TRUE(),
        _Converted >= 6 || _Inquiries >= 12, "High",
        _Converted >= 2 || _Inquiries >= 6,  "Medium",
        _Inquiries > 0  || _Converted > 0,   "Low",
        "Dormant"
    )

RETURN IF ( 'Accounts'[Status] = "Rotatable", "Dormant", _Base )
```

### Activity score

Pure recency, weighted toward conversion. Orders count for 80, inquiries for 20:

```dax
Activity Score =
VAR _DaysSinceInquiry = [Days Since Last Inquiry]
VAR _DaysSinceOrder   = [Days Since Last Order]
VAR _InquiryScore = IF ( NOT ISBLANK ( _DaysSinceInquiry ), MAX ( 0, 20 - ( _DaysSinceInquiry * 0.2 ) ), 0 )
VAR _OrderScore   = IF ( NOT ISBLANK ( _DaysSinceOrder ),   MAX ( 0, 80 - ( _DaysSinceOrder * 0.8 ) ),   0 )
RETURN MIN ( _InquiryScore + _OrderScore, 100 )
```

The 80/20 split reflects a business judgment: an account that inquires constantly but never buys is worth less attention than one that quietly places orders. Change the weights to change that judgment.

### Priority score

Status alone is a poor work queue — it treats a lapsed major account the same as one that never bought anything. This weights status by history:

```dax
Priority Score =
VAR _Status     = SELECTEDVALUE ( 'Accounts'[Status] )
VAR _Volume     = [Total Volume]
VAR _OrderCount = [Total Orders]
VAR _StrikeRate = [Strike Rate]
VAR _HasHistory = _OrderCount > 0

VAR _VolumeNormalizer = [Volume Normalizer]   -- parameter

VAR _Base =
    SWITCH (
        _Status,
        "Rotatable", IF ( _HasHistory, 100, 30 ),
        "At Risk",   IF ( _HasHistory, 70,  40 ),
        "Healthy",   20,
        "New",       10,
        "Reassigned", 5,
        0
    )

VAR _VolumeMultiplier = 1 + DIVIDE ( _Volume, _VolumeNormalizer, 0 )
VAR _ConversionBonus  = IF ( _HasHistory && _StrikeRate > 0.5, 1.3, 1 )

RETURN _Base * _VolumeMultiplier * _ConversionBonus
```

A rotatable account with real order history scores 100; one that never converted scores 30. That single distinction is what makes the queue usable.

### Rotation reason

Turns a status into an explanation:

```dax
Rotation Reason =
VAR _Status = SELECTEDVALUE ( 'Accounts'[Status] )
VAR _HasValidInquiry = [Has Valid Inquiry]      -- shared helper
RETURN
    IF (
        _Status = "Rotatable",
        SWITCH (
            TRUE(),
            NOT _HasValidInquiry, "No inquiry within window",
            _HasValidInquiry,     "Inquiry received, no conversion within window",
            "-"
        ),
        "-"
    )
```

### Volume trend

```dax
Volume Trend =
VAR _Current  = [Volume Last 90 Days]
VAR _Previous = [Volume Previous 90 Days]
VAR _Orders   = [Total Orders]
VAR _Age      = [Days Since Added]
VAR _Recent   = CALCULATE ( COUNTROWS ( Orders ), Orders[Order Date] >= TODAY() - 90 )
RETURN
SWITCH (
    TRUE(),
    _Orders <= 3 && _Age <= 90,                    "New",
    _Recent > 0 && ISBLANK ( _Previous ),          "Re-engaged",
    ISBLANK ( _Previous ) && ISBLANK ( _Current ), "Inactive",
    _Current > _Previous * 1.1,                    "Growing",
    _Current < _Previous * 0.9,                    "Declining",
    "Stable"
)
```

---

## Summary measures

| Measure | Definition |
|---|---|
| Total Accounts | `DISTINCTCOUNT` of account keys |
| Healthy / At Risk / Rotatable / New / Reassigned | Row counts per status |
| Strike Rate | `DIVIDE([Total Orders], [Total Inquiries])` |
| Volume at Risk | Volume attributable to rotatable accounts |
| Owner Rotation % | Rotatable ÷ total, per owner |
| Accounts Silent 60+ / 90+ Days | No activity in the period |

---

## Row-level security

| Role | Accounts | Metrics | Queue | Team comparison |
|---|---|---|---|---|
| Owner | Own only | Own only | Own only | Hidden |
| Manager | Team | Team aggregate | Team | Own team |
| Leadership | All | Full | All | All teams |

```dax
-- Role: Owner (filter on Owners table)
[Email] = USERPRINCIPALNAME()

-- Role: Manager — own accounts plus direct reports
[Email] = USERPRINCIPALNAME()
    || [Manager Email] = USERPRINCIPALNAME()

-- Role: Leadership — role-driven, no row filter
LOOKUPVALUE ( Owners[Role], Owners[Email], USERPRINCIPALNAME() ) IN { "Director", "VP" }
```

Because `Accounts` sits between `Owners` and both fact tables, the filter propagates to every visual automatically. One role definition secures the whole model.

---

## Alerting

```dax
Alert Level =
VAR _Status    = 'Accounts'[Status]
VAR _DaysLeft  = 'Accounts'[Days Until Rotation]
VAR _Volume    = CALCULATE ( SUM ( Orders[Volume] ), ALLEXCEPT ( 'Accounts', 'Accounts'[Account Key] ) )
VAR _HighValue = _Volume > [High Value Threshold]      -- parameter
VAR _Engagement = 'Accounts'[Engagement Level]
RETURN
SWITCH (
    TRUE(),
    _Status = "Rotatable" && _HighValue,                       "Critical",
    _Status = "At Risk"   && _HighValue,                       "Urgent",
    _Status IN { "At Risk", "Healthy" }
        && _DaysLeft <= 30 && _DaysLeft > 0,                   "Warning",
    _Engagement = "Dormant",                                   "Watch",
    BLANK()
)
```

Paired with Power Automate, this drives a weekly manager summary and a daily owner digest without anyone opening the report.

---

## Techniques demonstrated

- Multi-condition classification with `SWITCH ( TRUE() )` and layered VARs
- Calculated columns for filterable, sliceable status
- Inactive relationships and `USERELATIONSHIP()` for multiple fact tables on one date dimension
- Weighted scoring that combines categorical status with continuous history
- Dynamic RLS via `USERPRINCIPALNAME()` with manager hierarchy traversal
- Parameterized thresholds instead of hardcoded rules
- `ALLEXCEPT` for row-level aggregation inside a calculated column

---

## Stack

Power BI Desktop · DAX · Power Query (M) · star schema · dynamic RLS · optional Power Automate alerting

---

**Note:** This write-up documents a reusable modeling pattern. Table names, thresholds, and scoring weights are illustrative. No production dataset, schema, or business rule from any employer is reproduced here.
