# 🔒 RLS Implementation Guide

Row-Level Security (RLS) ensures users see only the data they're authorized to access. This guide covers strategy, implementation, testing, and governance.

---

## 🎯 RLS Strategy

### When to Use RLS

| Scenario | RLS Needed? | Alternative |
|----------|-------------|-------------|
| Traders should only see their customers | ✅ Yes | — |
| Managers see their team's data | ✅ Yes | — |
| Executives see everything | ✅ Yes (no filter) | — |
| Different reports for different regions | ⚠️ Maybe | Separate workspaces |
| Public dashboard, same data for all | ❌ No | — |
| Highly sensitive data (PII, salary) | ✅ Yes | Separate dataset |

### RLS Architecture Patterns

#### Pattern 1: Direct User Filter

User email directly matches data column.

```
User Table
├── Email: john@company.com
├── Email: jane@company.com
└── Email: mike@company.com
         │
         │ RLS Filter: [Email] = USERPRINCIPALNAME()
         ▼
    Filtered Data
```

**Best for:** Simple scenarios, small user base.

#### Pattern 2: Role-Based Hierarchy

Filter based on organizational hierarchy.

```
Trader Lookup
├── Trader: John (Manager: Jane)
├── Trader: Mike (Manager: Jane)
└── Trader: Jane (Manager: null) ← Manager
         │
         │ RLS: [Email] = UPN() OR [Manager Email] = UPN()
         ▼
    Filtered Data (Jane sees John + Mike + Self)
```

**Best for:** Team-based access, manager oversight.

#### Pattern 3: Dynamic Security Table

Separate table maps users to data access.

```
Security Table
├── User: john@company.com → Region: EMEA
├── User: john@company.com → Region: APAC
├── User: jane@company.com → Region: Americas
└── User: mike@company.com → Region: ALL
         │
         │ RLS on Security Table, relationship to Facts
         ▼
    Filtered Data
```

**Best for:** Complex access rules, multi-dimensional security.

---

## 🛠️ Implementation Steps

### Step 1: Design the Security Model

Document access requirements:

| Role | Customers | Deals | Financials | Reports |
|------|-----------|-------|------------|---------|
| Trader | Own only | Own only | Own only | Operations |
| Manager | Team | Team | Team | Operations + Team KPIs |
| Finance | All | All | All | Financial reports |
| Executive | All | All | All | All |

### Step 2: Create Security Reference Table

If not using direct filter, create a security table:

```
SalesTraderManagers
├── Dynamics User Name
├── Trader
├── Email ← For RLS lookup
├── Manager
├── Manager Email ← For hierarchy
├── Role ← For role-based logic
├── Office ← For regional filtering
└── Status (Active/Inactive)
```

### Step 3: Establish Relationships

Security table must connect to fact tables:

```
SalesTraderManagers ──1:M──► Deals
         │
         └──1:M──► Enquiries
         │
         └──1:M──► Customers
```

**Important:** RLS filters propagate through relationships. Plan your model accordingly.

### Step 4: Create Roles in Power BI Desktop

1. Go to **Modeling → Manage Roles**
2. Create roles matching your access matrix

#### Role: Trader
```dax
// Filter on SalesTraderManagers
[Email] = USERPRINCIPALNAME()
```

#### Role: Manager
```dax
// Filter on SalesTraderManagers
[Email] = USERPRINCIPALNAME()
    || [Manager Email] = USERPRINCIPALNAME()
```

#### Role: Leadership
```dax
// Option 1: No filter (sees everything)
// Leave blank

// Option 2: Filter by role
LOOKUPVALUE(
    SalesTraderManagers[Role],
    SalesTraderManagers[Email], USERPRINCIPALNAME()
) IN {"Director", "VP", "CEO"}
```

#### Role: Finance
```dax
// Finance sees all data
// Role exists for explicit assignment, no filter applied
TRUE()
```

### Step 5: Test Locally

1. Go to **Modeling → View as Roles**
2. Select role and enter test email
3. Verify correct data filtering
4. Test edge cases

### Step 6: Publish and Assign

1. Publish to Power BI Service
2. Go to **Dataset → Security**
3. Add users/groups to appropriate roles
4. Use Azure AD groups for scalability

---

## 🔐 Security Patterns

### Pattern: Office/Region-Based RLS

```dax
// User sees data for their office only
VAR _UserOffice = 
    LOOKUPVALUE(
        SalesTraderManagers[Office],
        SalesTraderManagers[Email], USERPRINCIPALNAME()
    )
RETURN
[Office] = _UserOffice || _UserOffice = "Global"
```

### Pattern: Cascading Hierarchy

```dax
// CEO → VP → Director → Manager → Trader
VAR _UserEmail = USERPRINCIPALNAME()
VAR _UserRole = 
    LOOKUPVALUE(
        SalesTraderManagers[Role],
        SalesTraderManagers[Email], _UserEmail
    )
RETURN
SWITCH(
    _UserRole,
    "CEO", TRUE(),  -- Sees all
    "VP", [Region] IN {"EMEA", "APAC"},  -- Sees specific regions
    "Director", [Manager Email] = _UserEmail || [Email] = _UserEmail,
    "Manager", [Manager Email] = _UserEmail || [Email] = _UserEmail,
    [Email] = _UserEmail  -- Default: own data only
)
```

### Pattern: Include Unassigned Records

Ensure orphan records are visible to appropriate users:

```dax
// Trader sees own customers + unassigned
[Trader Email] = USERPRINCIPALNAME()
    || ISBLANK([Trader Email])
    || [Trader] = "Unassigned"
```

### Pattern: Time-Based Access

```dax
// User can only see current year + last year
VAR _CurrentYear = YEAR(TODAY())
RETURN
[Year] >= _CurrentYear - 1
```

### Pattern: Exclude Sensitive Records

```dax
// All users, but hide confidential deals
[Email] = USERPRINCIPALNAME()
    && [Confidential Flag] <> TRUE()
```

---

## ✅ Testing Checklist

### Pre-Deployment Testing

| Test | Description | Pass? |
|------|-------------|-------|
| Trader sees own data only | Filter by individual trader | ☐ |
| Trader cannot see peer data | Check adjacent trader's data | ☐ |
| Manager sees team data | All direct reports visible | ☐ |
| Manager cannot see other teams | Check sibling manager's team | ☐ |
| Executive sees all | No filtering applied | ☐ |
| Unassigned records visible | Orphan data accessible | ☐ |
| New user defaults correct | No role = no access | ☐ |
| Performance acceptable | Large dataset filter speed | ☐ |

### Post-Deployment Testing

| Test | Description | Pass? |
|------|-------------|-------|
| Users can access report | Basic access works | ☐ |
| Correct data displayed | Spot check with users | ☐ |
| Export respects RLS | Excel export filtered | ☐ |
| Analyze in Excel respects RLS | XMLA connection filtered | ☐ |
| Scheduled refresh works | Data refreshes correctly | ☐ |

### Edge Cases to Verify

- [ ] User with no role assignment (should see nothing or error)
- [ ] User in multiple roles (most permissive wins)
- [ ] User with special characters in email
- [ ] Guest users (B2B scenarios)
- [ ] Service accounts (for scheduled refresh)

---

## 🔍 Auditing & Monitoring

### Access Review Process

| Frequency | Action |
|-----------|--------|
| Monthly | Review role membership |
| Quarterly | Audit access vs. job function |
| On termination | Remove user from all roles |
| On role change | Update role membership |

### Power BI Activity Log

Monitor for anomalies:
- Unusual data export volumes
- Access outside business hours
- Failed access attempts
- Role membership changes

### Documentation Requirements

Maintain for each RLS-protected dataset:

```
RLS Documentation - [Dataset Name]
├── Access Matrix (Role → Data Access)
├── Role Definitions (DAX code)
├── User/Group Assignments
├── Testing Evidence
├── Approval Sign-off
└── Review History
```

---

## ⚠️ Common Mistakes

### 1. ❌ Not Testing with Actual Users

**Problem:** Works in "View as Role" but fails in production.
**Solution:** Test with actual user accounts before go-live.

### 2. ❌ Forgetting Relationship Direction

**Problem:** RLS doesn't filter related tables.
**Solution:** Verify relationships flow from security table to fact tables.

### 3. ❌ Case Sensitivity

**Problem:** `USERPRINCIPALNAME()` returns lowercase; data has mixed case.
**Solution:** 
```dax
LOWER([Email]) = USERPRINCIPALNAME()
```

### 4. ❌ Hard-coded User Lists

**Problem:** DAX contains list of users that must be updated manually.
**Solution:** Use security table with user-to-role mapping.

### 5. ❌ No Default Deny

**Problem:** Users without role assignment see all data.
**Solution:** Ensure no default access; require explicit role assignment.

### 6. ❌ Performance Degradation

**Problem:** RLS filter causes slow report loads.
**Solution:** 
- Optimize security table
- Use direct column filters, not LOOKUPVALUE in filter expression
- Consider pre-filtered datasets for very large scenarios

---

## 📋 RLS Governance Checklist

### Before Deployment
- [ ] Access requirements documented and approved
- [ ] Security table designed and populated
- [ ] Roles created and tested
- [ ] Edge cases verified
- [ ] Performance validated
- [ ] Documentation complete

### At Deployment
- [ ] Users/groups assigned to roles
- [ ] Initial access verification with users
- [ ] Monitoring enabled

### Ongoing
- [ ] Monthly access review
- [ ] Quarterly audit
- [ ] Termination process integrated with HR
- [ ] New user onboarding includes BI access request
- [ ] Changes logged and approved

---

## 📚 Reference

### Useful Functions

| Function | Purpose |
|----------|---------|
| `USERPRINCIPALNAME()` | Returns current user's email |
| `USERNAME()` | Returns domain\username (legacy) |
| `LOOKUPVALUE()` | Get value from security table |
| `PATHCONTAINS()` | Check parent-child hierarchy |
| `CUSTOMDATA()` | Access connection string parameter |

### External Resources

- [Microsoft RLS Documentation](https://docs.microsoft.com/power-bi/admin/service-admin-rls)
- [Power BI Security Whitepaper](https://docs.microsoft.com/power-bi/guidance/whitepaper-powerbi-security)
