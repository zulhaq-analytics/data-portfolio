# 🧮 DAX Best Practices

Standards for writing readable, performant, and maintainable DAX across all Power BI projects.

---

## 📝 Formatting Standards

### Use Variables (VAR)

Variables improve readability and performance by calculating values once.

**❌ Bad:**
```dax
Profit Margin = 
DIVIDE(
    SUM(Deals[Sales Revenue]) - SUM(Deals[Purchase Cost]),
    SUM(Deals[Sales Revenue])
)
```

**✅ Good:**
```dax
Profit Margin = 
VAR _Revenue = SUM(Deals[Sales Revenue])
VAR _Cost = SUM(Deals[Purchase Cost])
VAR _Profit = _Revenue - _Cost
RETURN
DIVIDE(_Profit, _Revenue)
```

### Variable Naming

| Convention | Example |
|------------|---------|
| Prefix with underscore | `_Revenue`, `_Today`, `_CurrentDate` |
| Descriptive names | `_DaysSinceLastDeal`, `_CustomerCount` |
| No spaces | `_TotalVolume` not `_Total Volume` |

### Indentation & Line Breaks

- One argument per line for functions with multiple arguments
- Indent nested functions
- Align similar elements

**❌ Bad:**
```dax
Running Total = CALCULATE(SUM(Paid[Amount]),REMOVEFILTERS('Calendar Table'),KEEPFILTERS('Calendar Table'[Date]<=MAX('Calendar Table'[Date])))
```

**✅ Good:**
```dax
Running Total = 
VAR _CurrentDate = MAX('Calendar Table'[Date])
RETURN
CALCULATE(
    SUM(Paid[Amount]),
    REMOVEFILTERS('Calendar Table'),
    KEEPFILTERS('Calendar Table'[Date] <= _CurrentDate)
)
```

---

## ⚡ Performance Patterns

### 1. Measures vs. Calculated Columns

| Use Measure When... | Use Calculated Column When... |
|---------------------|-------------------------------|
| Value depends on filter context | Value is fixed per row |
| Aggregation is needed | Used in slicers or filters |
| Value changes based on selection | Used in relationships |
| Most calculations | Sorting by calculated value |

**Rule of thumb:** Start with measures. Only use calculated columns when you can't achieve the result with a measure.

### 2. CALCULATE vs. FILTER

**❌ Avoid FILTER on large tables:**
```dax
Revenue 2024 = 
CALCULATE(
    SUM(Deals[Sales Revenue]),
    FILTER(Deals, Deals[Year] = 2024)  -- Iterates entire table
)
```

**✅ Use direct column filter:**
```dax
Revenue 2024 = 
CALCULATE(
    SUM(Deals[Sales Revenue]),
    Deals[Year] = 2024  -- Uses index
)
```

**When FILTER is appropriate:**
- Complex conditions that can't be expressed as column filters
- Filtering based on measure results
- OR conditions across multiple columns

### 3. SWITCH vs. Nested IF

**❌ Avoid nested IF:**
```dax
Status = 
IF([Days] <= 0, "Overdue",
    IF([Days] <= 30, "At Risk",
        IF([Days] <= 60, "Warning", "Healthy")))
```

**✅ Use SWITCH + TRUE():**
```dax
Status = 
SWITCH(
    TRUE(),
    [Days] <= 0, "Overdue",
    [Days] <= 30, "At Risk",
    [Days] <= 60, "Warning",
    "Healthy"
)
```

### 4. Avoid Repeated Calculations

**❌ Same calculation repeated:**
```dax
Metric = 
IF(
    SUM(Deals[Volume]) > 1000,
    SUM(Deals[Volume]) * 1.1,
    SUM(Deals[Volume]) * 0.9
)
```

**✅ Calculate once with VAR:**
```dax
Metric = 
VAR _Volume = SUM(Deals[Volume])
RETURN
IF(_Volume > 1000, _Volume * 1.1, _Volume * 0.9)
```

---

## 🔄 Context Transition Patterns

### CALCULATE Fundamentals

CALCULATE does two things:
1. Changes filter context
2. Triggers context transition (row context → filter context)

### REMOVEFILTERS vs. ALL

| Function | Use When |
|----------|----------|
| `REMOVEFILTERS(Table)` | Clear all filters on a table |
| `REMOVEFILTERS(Column)` | Clear filter on specific column |
| `ALL(Table)` | Same as REMOVEFILTERS, but also usable in FILTER |
| `ALLEXCEPT(Table, Column)` | Keep only specified column filters |
| `ALLSELECTED()` | Respect slicer selections in visuals |

**Example — Running Total:**
```dax
Running Paid = 
VAR _CurrentDate = MAX('Calendar Table'[Date])
RETURN
CALCULATE(
    SUM(Paid[Amount]),
    REMOVEFILTERS('Calendar Table'),  -- Clear date filters
    KEEPFILTERS('Calendar Table'[Date] <= _CurrentDate)  -- Apply new filter
)
```

### KEEPFILTERS

Preserves existing filter context while adding new conditions:

```dax
High Value Deals = 
CALCULATE(
    COUNT(Deals[Deal ID]),
    KEEPFILTERS(Deals[Volume] > 10000)  -- Adds to existing filters
)
```

---

## 🔗 Relationship Patterns

### TREATAS for Virtual Relationships

When tables aren't directly related:

```dax
Allocated Cash = 
CALCULATE(
    SUM(CashAllocation[Amount]),
    TREATAS(
        VALUES(SalesTraderManagers[Office]),
        CashAllocation[Office]
    )
)
```

### USERELATIONSHIP for Inactive Relationships

Activate inactive relationships in specific measures:

```dax
Revenue by Invoice Date = 
CALCULATE(
    SUM(Deals[Sales Revenue]),
    USERELATIONSHIP(Deals[Invoice Due Date], 'Calendar Table'[Date])
)
```

### RELATED and RELATEDTABLE

| Function | Direction | Returns |
|----------|-----------|---------|
| `RELATED()` | Many → One | Single value from related table |
| `RELATEDTABLE()` | One → Many | Table of related rows |

---

## 📊 Common Patterns Library

### Percentage of Total

```dax
% of Total = 
VAR _CurrentValue = [Total Revenue]
VAR _TotalValue = CALCULATE([Total Revenue], REMOVEFILTERS(Customer[Customer Name]))
RETURN
DIVIDE(_CurrentValue, _TotalValue)
```

### Year-over-Year Growth

```dax
Revenue YoY % = 
VAR _Current = [Total Revenue]
VAR _PriorYear = 
    CALCULATE(
        [Total Revenue],
        SAMEPERIODLASTYEAR('Calendar Table'[Date])
    )
RETURN
DIVIDE(_Current - _PriorYear, _PriorYear)
```

### Running Total

```dax
Running Total = 
VAR _CurrentDate = MAX('Calendar Table'[Date])
RETURN
CALCULATE(
    [Total Revenue],
    REMOVEFILTERS('Calendar Table'),
    'Calendar Table'[Date] <= _CurrentDate
)
```

### Ranking

```dax
Customer Rank = 
RANKX(
    ALL(Customer[Customer Name]),
    [Total Revenue],
    ,
    DESC,
    DENSE
)
```

### Dynamic Segmentation

```dax
Customer Segment = 
VAR _Revenue = [Total Revenue]
RETURN
SWITCH(
    TRUE(),
    _Revenue >= 1000000, "Enterprise",
    _Revenue >= 100000, "Mid-Market",
    _Revenue >= 10000, "SMB",
    "Small"
)
```

### COALESCE for Fallback Values

```dax
Display Value = 
COALESCE(
    [Primary Metric],
    [Secondary Metric],
    0
)
```

### Conditional Aggregation

```dax
Active Customer Count = 
CALCULATE(
    DISTINCTCOUNT(Customer[Customer Key]),
    Customer[Status] = "Active"
)
```

---

## ⚠️ Anti-Patterns to Avoid

### 1. ❌ Scalar in Iterator Context

```dax
-- BAD: Calculates Total Revenue for every row
Bad Measure = SUMX(Deals, [Total Revenue] * Deals[Quantity])

-- GOOD: Use column reference
Good Measure = SUMX(Deals, Deals[Price] * Deals[Quantity])
```

### 2. ❌ Filtering in Aggregation

```dax
-- BAD: FILTER iterates entire table
Bad = CALCULATE(SUM(Deals[Amount]), FILTER(ALL(Deals), Deals[Year] = 2024))

-- GOOD: Direct filter
Good = CALCULATE(SUM(Deals[Amount]), Deals[Year] = 2024)
```

### 3. ❌ Unnecessary CALCULATE

```dax
-- BAD: CALCULATE with no filter modification
Bad = CALCULATE(SUM(Deals[Amount]))

-- GOOD: Simple aggregation
Good = SUM(Deals[Amount])
```

### 4. ❌ Hard-coded Values

```dax
-- BAD: Hard-coded threshold
Bad = IF([Revenue] > 1000000, "High", "Low")

-- GOOD: Use parameter table or variable
Good = 
VAR _Threshold = SELECTEDVALUE(Parameters[High Value Threshold], 1000000)
RETURN IF([Revenue] > _Threshold, "High", "Low")
```

### 5. ❌ Complex Nested Measures Without Comments

```dax
-- BAD: No explanation
Complex = CALCULATE(SUMX(...), FILTER(...), KEEPFILTERS(...))

-- GOOD: With comments
Complex = 
-- Calculate weighted average excluding cancelled deals
VAR _ValidDeals = FILTER(Deals, Deals[Status] <> "Cancelled")
...
```

---

## 📋 Code Review Checklist

Before approving any DAX:

### Readability
- [ ] Variables used for repeated calculations
- [ ] Descriptive variable names with underscore prefix
- [ ] Proper indentation and line breaks
- [ ] Comments for complex logic

### Performance
- [ ] No FILTER on large tables without necessity
- [ ] Measures preferred over calculated columns
- [ ] No repeated calculations
- [ ] SWITCH used instead of nested IF

### Correctness
- [ ] Handles blank/null values appropriately
- [ ] DIVIDE used instead of `/` operator
- [ ] Context transition understood and intended
- [ ] Tested with edge cases

### Maintainability
- [ ] No magic numbers (use variables or parameters)
- [ ] Follows team naming conventions
- [ ] Description added to measure
- [ ] Logic documented if complex

---

## 📚 Reference

### Function Categories

| Category | Key Functions |
|----------|---------------|
| Aggregation | SUM, COUNT, AVERAGE, MIN, MAX, DISTINCTCOUNT |
| Filter | CALCULATE, FILTER, ALL, REMOVEFILTERS, KEEPFILTERS |
| Table | SUMMARIZE, ADDCOLUMNS, SELECTCOLUMNS, UNION |
| Iterator | SUMX, COUNTX, AVERAGEX, MAXX, MINX |
| Relationship | RELATED, RELATEDTABLE, TREATAS, USERELATIONSHIP |
| Time Intelligence | SAMEPERIODLASTYEAR, DATEADD, DATESYTD, TOTALYTD |
| Logical | IF, SWITCH, AND, OR, COALESCE, IFERROR |
| Information | ISBLANK, HASONEVALUE, SELECTEDVALUE, VALUES |
