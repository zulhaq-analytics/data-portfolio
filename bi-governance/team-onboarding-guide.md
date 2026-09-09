# 👋 Team Onboarding Guide

Welcome to the BI team! This guide will help you get up to speed with our standards, tools, and workflows.

---

## 🚀 First Week Checklist

### Day 1: Access & Setup
- [ ] Power BI Desktop installed (latest version)
- [ ] Power BI Service access granted
- [ ] Access to development workspace
- [ ] SharePoint/data source credentials
- [ ] Git repository access (if applicable)
- [ ] Teams channel added

### Day 2-3: Standards Review
- [ ] Read [Data Modeling Standards](./data-modeling-standards.md)
- [ ] Read [DAX Best Practices](./dax-best-practices.md)
- [ ] Review [RLS Implementation Guide](./rls-implementation-guide.md)
- [ ] Explore 2-3 existing production reports

### Day 4-5: Hands-On
- [ ] Clone an existing report as practice
- [ ] Make a small modification
- [ ] Get first code review
- [ ] Shadow a deployment

---

## 📁 Project Structure

Every Power BI project should follow this structure:

```
project-name/
├── README.md                 # Project documentation
├── project-name.pbix         # Power BI file
├── screenshots/              # Dashboard screenshots
│   └── dashboard-preview.png
├── data/                     # Sample/test data (if applicable)
│   └── sample-data.csv
└── docs/                     # Additional documentation
    └── data-dictionary.md
```

---

## 📝 Naming Conventions Quick Reference

### Tables
| Type | Format | Example |
|------|--------|---------|
| Fact | Plural PascalCase | `Deals`, `Payments` |
| Dimension | Descriptive PascalCase | `Customer Lookup`, `Calendar Table` |
| Measures | "Measures Table" | `Measures Table` |

### Columns
| Type | Format | Example |
|------|--------|---------|
| Keys | Entity + Key/ID | `Customer Key`, `Deal ID` |
| Dates | Noun + Date | `Deal Date`, `Invoice Due Date` |
| Flags | Is + Condition | `Is Active`, `Is Rotatable` |

### Measures
| Type | Format | Example |
|------|--------|---------|
| Totals | Total/Sum + Noun | `Total Revenue`, `Deal Count` |
| Ratios | Metric + % | `Strike Rate`, `GM %` |
| Running | Running + Metric | `Running Paid`, `Running Utilization` |

---

## 🧮 DAX Quick Reference

### Must-Know Patterns

**Simple Aggregation:**
```dax
Total Revenue = SUM(Deals[Sales Revenue])
```

**Safe Division:**
```dax
Profit Margin = DIVIDE([Gross Profit], [Total Revenue])
```

**Conditional Count:**
```dax
Active Customers = 
CALCULATE(
    DISTINCTCOUNT(Customers[Customer Key]),
    Customers[Status] = "Active"
)
```

**Running Total:**
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

**SWITCH for Classification:**
```dax
Status = 
SWITCH(
    TRUE(),
    [Days] <= 0, "Overdue",
    [Days] <= 30, "At Risk",
    "Healthy"
)
```

### Variables Template
```dax
Measure Name = 
VAR _Variable1 = [Some Calculation]
VAR _Variable2 = [Another Calculation]
VAR _Result = _Variable1 + _Variable2
RETURN
_Result
```

---

## ✅ Code Review Checklist

Before requesting a review, verify:

### Structure
- [ ] Star schema followed
- [ ] Relationships documented
- [ ] No unnecessary tables/columns

### Naming
- [ ] Tables follow conventions
- [ ] Columns follow conventions
- [ ] Measures follow conventions

### DAX
- [ ] Variables used for clarity
- [ ] DIVIDE() instead of `/`
- [ ] No hard-coded values
- [ ] SWITCH instead of nested IF

### Documentation
- [ ] README updated
- [ ] Measure descriptions added
- [ ] Complex logic commented

---

## 🔄 Development Workflow

```
1. CREATE      ──►  2. DEVELOP   ──►  3. REVIEW    ──►  4. DEPLOY
   Branch            Local PBIX       Code Review       Publish
   (if using Git)    Development      Standards Check   to Service
```

### Step 1: Understand Requirements
- Meet with stakeholder
- Document KPIs and filters needed
- Identify data sources
- Sketch wireframe

### Step 2: Build Data Model
- Import/connect to data
- Design star schema
- Create relationships
- Add date table

### Step 3: Create Measures
- Start with base aggregations
- Add calculated metrics
- Follow DAX standards
- Add descriptions

### Step 4: Build Visuals
- Create page layout
- Add KPI cards first
- Build main visuals
- Add filters/slicers

### Step 5: Review & Test
- Self-review against checklist
- Request peer review
- Test with sample data
- Verify RLS (if applicable)

### Step 6: Deploy
- Publish to workspace
- Configure refresh schedule
- Set up alerts (if needed)
- Notify stakeholders

---

## 🚫 Common Mistakes to Avoid

### Data Modeling
| ❌ Don't | ✅ Do |
|---------|------|
| Single flat table | Star schema |
| Import all columns | Only import needed columns |
| Create multiple date tables | Single shared date table |
| Use bi-directional relationships everywhere | Single direction unless required |

### DAX
| ❌ Don't | ✅ Do |
|---------|------|
| `[Revenue] / [Cost]` | `DIVIDE([Revenue], [Cost])` |
| Nested IF statements | SWITCH + TRUE() |
| Repeat calculations | Use variables (VAR) |
| Hard-code values | Use parameters or variables |

### Visuals
| ❌ Don't | ✅ Do |
|---------|------|
| Too many visuals per page | 5-7 max per page |
| Rainbow color schemes | Consistent brand colors |
| 3D charts | Simple 2D charts |
| Pie charts with many slices | Bar charts for comparison |

---

## 🛠️ Useful Tools

### Power BI Desktop Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl + Enter` | Execute DAX in formula bar |
| `Ctrl + Shift + D` | Duplicate page |
| `Ctrl + Click` | Multi-select visuals |
| `Alt + F4` | Close (save first!) |

### External Tools
| Tool | Purpose |
|------|---------|
| DAX Studio | Debug and optimize DAX |
| Tabular Editor | Bulk edit model metadata |
| ALM Toolkit | Compare/deploy models |
| Power BI Helper | Document models |

### Learning Resources
| Resource | What You'll Learn |
|----------|-------------------|
| SQLBI.com | Advanced DAX patterns |
| Guy in a Cube (YouTube) | Practical tutorials |
| Microsoft Learn | Official certifications |
| Power BI Community | Q&A and forums |

---

## ❓ Getting Help

### Before Asking
1. Check existing documentation
2. Search Power BI Community
3. Try debugging with DAX Studio
4. Review similar patterns in existing reports

### How to Ask
When posting a question:
```
📌 What I'm trying to do:
[Clear description of goal]

📌 What I've tried:
[DAX or steps you've attempted]

📌 What's happening:
[Error message or unexpected result]

📌 Sample data:
[If relevant, include sample rows]
```

### Who to Ask
| Topic | Contact |
|-------|---------|
| DAX help | [Senior BI Developer] |
| Data access issues | [BI Admin] |
| Business requirements | [BI Manager] |
| Infrastructure/deployment | [IT Support] |

---

## 📅 30-60-90 Day Goals

### First 30 Days
- [ ] Complete all onboarding tasks
- [ ] Understand 3 key production reports
- [ ] Make first contribution (bug fix or small enhancement)
- [ ] Pass internal DAX quiz

### 30-60 Days
- [ ] Own your first small project end-to-end
- [ ] Conduct a peer code review
- [ ] Document one undocumented report
- [ ] Propose one improvement to standards

### 60-90 Days
- [ ] Lead a medium complexity project
- [ ] Train another team member on one topic
- [ ] Contribute to governance documentation
- [ ] Identify one process improvement

---

## 🎯 Success Metrics

You're doing well when:
- ✅ Reports pass code review on first submission
- ✅ Stakeholders request you for their projects
- ✅ You can explain your DAX choices
- ✅ You help others troubleshoot
- ✅ You suggest improvements to standards

---

Welcome aboard! 🚀
