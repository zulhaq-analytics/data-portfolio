# 📐 Data Modeling Standards

Consistent data modeling ensures reports are maintainable, performant, and understandable across teams.

---

## 🏗️ Schema Design

### Star Schema First

All models should follow star schema principles:

```
        ┌─────────────┐
        │  Dimension  │
        │   Tables    │
        └──────┬──────┘
               │ 1
               │
               │ M
        ┌──────┴──────┐
        │    Fact     │
        │   Tables    │
        └──────┬──────┘
               │ M
               │
               │ 1
        ┌──────┴──────┐
        │  Dimension  │
        │   Tables    │
        └─────────────┘
```

| Table Type | Purpose | Examples |
|------------|---------|----------|
| **Fact** | Transactional data with metrics | Deals, Payments, Enquiries |
| **Dimension** | Descriptive attributes for filtering/grouping | Customers, Traders, Products, Ports |
| **Bridge** | Many-to-many relationship resolution | Customer-Product mapping |
| **Calculated** | DAX-generated reference tables | Dynamic Thresholds, Date Spine |

---

## 📝 Naming Conventions

### Tables

| Type | Convention | Example |
|------|------------|---------|
| Fact tables | Plural noun, PascalCase | `Deals`, `Payments`, `Enquiries` |
| Dimension tables | Singular or descriptive, PascalCase | `Customer Lookup`, `Trader Lookup`, `Calendar Table` |
| Bridge tables | Purpose + "Bridge" or "Mapping" | `Customer Product Mapping` |
| Measure tables | "Measures Table" or "__ Measures" | `Measures Table` |
| Calculated tables | Descriptive name | `Dynamic Thresholds`, `Customer Overdue Summary` |

**Avoid:**
- ❌ Abbreviations: `Cust`, `Txn`, `Inv`
- ❌ Prefixes like `tbl_`, `dim_`, `fact_` (Power BI isn't a database)
- ❌ Source system names: `SAP_VBAK`, `BC_Sales`

### Columns

| Type | Convention | Example |
|------|------------|---------|
| Keys | Entity + "Key" or "ID" | `Customer Key`, `Deal ID` |
| Dates | Descriptive + "Date" | `Deal Date`, `Invoice Due Date`, `Reassignment Date` |
| Amounts | Descriptive noun | `Purchase Cost`, `Sales Revenue`, `Gross Profit` |
| Counts | Noun (will be aggregated) | `Volume`, `Quantity` |
| Flags | "Is" + Condition | `Is Active`, `Is Rotatable` |
| Calculated columns | Descriptive of logic | `Unified Status`, `Days Until Rotation`, `Alert Level` |

**Avoid:**
- ❌ Spaces at start/end
- ❌ Special characters: `#`, `@`, `/`
- ❌ Reserved words: `Date`, `Year`, `Month` (use `Deal Date`, `Calendar Year`)

### Measures

| Type | Convention | Example |
|------|------------|---------|
| Aggregations | Verb + Noun or just Noun | `Total Revenue`, `Deal Count`, `Volume Fixed` |
| Ratios/Percentages | Metric + "%" or "Rate" | `Strike Rate`, `GM %`, `Cash Utilization %` |
| Running totals | "Running" + Metric | `Running Paid`, `Running Utilization` |
| Time comparisons | Metric + Period | `Revenue YTD`, `Volume Last 90 Days` |
| Rankings | Metric + "Rank" | `Gross Profit Rank` |
| Conditional | Descriptive of condition | `Overdue Receivables`, `High Priority Alerts` |

---

## 🔗 Relationship Rules

### Cardinality

| From | To | When to Use |
|------|-----|-------------|
| Many | One | Standard fact-to-dimension (most common) |
| One | One | Rare; usually indicates modeling issue |
| Many | Many | Avoid; use bridge table instead |

### Cross-Filter Direction

| Setting | When to Use |
|---------|-------------|
| **Single** | Default for most relationships |
| **Both** | Only when bidirectional filtering is required (e.g., many-to-many patterns) |

⚠️ **Warning:** Bidirectional filters can cause ambiguity and performance issues. Document why bidirectional is needed.

### Active vs. Inactive Relationships

| Type | Use Case |
|------|----------|
| **Active** | Primary relationship used by default |
| **Inactive** | Alternative date relationships, activated via USERELATIONSHIP() |

**Example:** Deals table has `Deal Date`, `Invoice Due Date`, `Payment Date` — only one can be active to Calendar Table. Others are inactive and activated in specific measures.

---

## 📁 Folder Organization

### Display Folders for Measures

Group measures logically in the Fields pane:

```
Measures Table
├── 📁 Revenue
│   ├── Total Revenue
│   ├── Revenue YTD
│   └── Revenue vs Budget
├── 📁 Profitability
│   ├── Gross Profit
│   ├── GM %
│   └── ROI
├── 📁 Operations
│   ├── Deal Count
│   ├── Strike Rate
│   └── Volume Fixed
└── 📁 Alerts
    ├── Critical Alerts
    ├── Total Active Alerts
    └── Alert Summary
```

### Hide Technical Columns

Hide from report view:
- ✅ Key columns used only for relationships
- ✅ Helper columns for calculated columns
- ✅ Deprecated columns kept for backward compatibility

Do NOT hide:
- ❌ Columns users need for filtering/slicing
- ❌ Date columns

---

## 📅 Date Table Requirements

Every model MUST have a dedicated date table:

### Required Columns

| Column | Purpose |
|--------|---------|
| `Date` | Primary key, continuous date range |
| `Year` | 4-digit year |
| `Month` | Month number (1-12) |
| `Month Name` | Full month name |
| `Quarter` | Q1, Q2, Q3, Q4 |
| `Day of Week` | Monday, Tuesday, etc. |
| `Is Weekend` | TRUE/FALSE |
| `Is Current Month` | TRUE/FALSE |
| `Is Current Year` | TRUE/FALSE |

### Optional (Business-Specific)

| Column | Purpose |
|--------|---------|
| `Fiscal Year` | If differs from calendar |
| `Fiscal Quarter` | If differs from calendar |
| `Is Working Day` | Excludes holidays |
| `Week Number` | ISO or custom |

### Mark as Date Table

Always mark the date table in Power BI:
1. Select the Date Table
2. Table Tools → Mark as Date Table
3. Select the Date column

---

## ✅ Pre-Deployment Checklist

Before publishing any model:

### Structure
- [ ] Star schema followed (no snowflake without justification)
- [ ] All relationships documented
- [ ] No circular dependencies
- [ ] Bidirectional filters justified and documented

### Naming
- [ ] Tables follow naming conventions
- [ ] Columns follow naming conventions
- [ ] Measures follow naming conventions
- [ ] No abbreviations or cryptic names

### Performance
- [ ] Calculated columns used only when necessary
- [ ] No unnecessary columns imported
- [ ] Date table marked as date table
- [ ] Large text columns evaluated for removal

### Documentation
- [ ] Data dictionary completed
- [ ] Measure descriptions added
- [ ] Relationship diagram saved
- [ ] RLS requirements documented

---

## 📚 Examples from Production Models

### Good: Account Rotation Model

```
Tables:
├── Customers Lookup (Dimension)
├── Deals (Fact)
├── Enquiries (Fact)
├── Trader Lookup (Dimension)
├── Reassignment Table (Fact)
├── Calendar Table (Dimension)
└── Measures Table (Measures)

Relationships:
├── Deals[Customer Key] → Customers Lookup[Customer Key] (M:1)
├── Enquiries[Customer Key] → Customers Lookup[Customer Key] (M:1)
├── Customers Lookup[Trader] → Trader Lookup[Dynamics User Name] (M:1)
└── Deals[Deal Date] → Calendar Table[Date] (M:1, Inactive)
```

### Good: Cash Utilization Model

```
Tables:
├── Deals Table (Central Dimension)
├── Payables (Fact - expected outflows)
├── Receivable (Fact - expected inflows)
├── Paid (Fact - actual outflows)
├── Receipts (Fact - actual inflows)
├── CashAllocation (Dimension)
├── SalesTraderManagers (Dimension + RLS)
├── Calendar Table (Dimension)
└── Customer Overdue Summary (Calculated Table)
```

---

## ⚠️ Common Mistakes to Avoid

| Mistake | Problem | Solution |
|---------|---------|----------|
| Single flat table | Poor performance, no filtering flexibility | Split into star schema |
| Too many calculated columns | Slow refresh, bloated model | Use measures where possible |
| Date columns not in date table | Time intelligence won't work | Create proper date table |
| Ambiguous relationships | Unexpected filter behavior | Review relationship direction |
| Importing all columns | Large model size | Import only needed columns |
| No measure descriptions | Users don't understand metrics | Add descriptions to all measures |
