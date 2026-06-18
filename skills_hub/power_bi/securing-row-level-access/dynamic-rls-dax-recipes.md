# Dynamic RLS DAX Recipes

Copy-paste DAX rule expressions for dynamic RLS. Each rule goes in **Manage roles** in Power BI Desktop, on the named table. After publishing, map a security group to the role in the service. Test every recipe with **View As** (see role-testing-view-as-checklist.md).

## Contents
- Recipe 1: Match the current user by UPN
- Recipe 2: Match by USERNAME (legacy)
- Recipe 3: Harden against unexpected identities (test each value)
- Recipe 4: Manager / org hierarchy via a path security table
- Recipe 5: Manager / org hierarchy via PATH() on a self-referencing dimension
- Recipe 6: Partial RLS — "% of all" without leaking other rows
- Recipe 7: Embedding with CUSTOMDATA()
- Recipe 8: Diagnostic "Who Am I" measure

---

## Recipe 1: Match the current user by UPN

Most common dynamic pattern. A hidden `Salesperson` (security) dimension stores one row per user with their UPN/email and the attribute that drives filtering (e.g. `Region`). The relationship `Salesperson[Region]` → fact propagates the filter.

**Rule on `Salesperson`:**
```dax
[EmailAddress] = USERPRINCIPALNAME()
```

The matched user's region flows to `Sales` through the model relationship — do **not** use `LOOKUPVALUE`; rely on the active relationship. If the security table is many-to-many with the fact (via a region bridge), enable **Apply security filter in both directions** on the relationship.

---

## Recipe 2: Match by USERNAME (legacy)

Use only when the model stores AD-style names rather than UPNs. Format can differ between Desktop preview and the service — test both.

**Rule on the security table:**
```dax
[UserLogin] = USERNAME()
```

---

## Recipe 3: Harden against unexpected identities (test each value)

When a rule branches on a passed role/value (embedding, or `USERNAME`), never let an unexpected value fall through to "all rows." Test each expected value explicitly; the final fallback returns **no** rows.

**Unsafe** (any value except `"Worker"` returns all rows):
```dax
IF(
    USERNAME() = "Worker",
    [Type] = "Internal",
    TRUE()
)
```

**Safe** (unexpected value → no rows):
```dax
IF(
    USERNAME() = "Worker",
    [Type] = "Internal",
    IF(
        USERNAME() = "Manager",
        TRUE(),
        FALSE()
    )
)
```

---

## Recipe 4: Manager / org hierarchy via a path security table

Goal: a manager sees their own rows **plus** all subordinates' rows. Build a **bridge/security table** that lists, for every employee, every manager above them (the ancestor set). This is a dimension in Kimball terms (cross-ref `conformed-dimension-management`): conform the employee key, treat the bridge as a many-to-many helper.

Security table `UserScope` grain = one row per (ManagerUPN, EmployeeKey-they-can-see). The fact relates to `Employee` on `EmployeeKey`; `UserScope` relates to `Employee` on the same key.

**Rule on `UserScope`:**
```dax
[ManagerUPN] = USERPRINCIPALNAME()
```

Filtering `UserScope` by the current manager leaves only the employee keys that manager may see; that set propagates to the fact. Because the path goes UserScope → Employee → Fact against the normal direction, enable **Apply security filter in both directions** on the UserScope↔Employee relationship and verify with View As.

Populate `UserScope` in Power Query/ETL by expanding each employee's manager chain (one row per ancestor, plus a self row so managers see their own data).

---

## Recipe 5: Manager / org hierarchy via PATH() on a self-referencing dimension

When the `Employee` dimension stores `EmployeeKey` and `ManagerKey` (parent-child), build a calculated column holding the ancestor path, then filter by whether the current user appears in each row's path.

**Calculated column on `Employee`** (one-time model setup, not a rule):
```dax
ManagerPath = PATH(Employee[EmployeeKey], Employee[ManagerKey])
```

**Calculated column** mapping the current user to their key is avoided in the rule itself; instead store `EmployeeUPN` on `Employee` and use a rule that keeps any row whose ancestor path contains the current user's key. Resolve the current user's key with a variable:

**Rule on `Employee`:**
```dax
VAR CurrentUserKey =
    LOOKUPVALUE(
        Employee[EmployeeKey],
        Employee[EmployeeUPN], USERPRINCIPALNAME()
    )
RETURN
    PATHCONTAINS(Employee[ManagerPath], CurrentUserKey)
```

Note: `LOOKUPVALUE` here resolves the *current user's own key* (it is not substituting for a relationship that propagates the filter), so it is acceptable. `PATHCONTAINS` returns `TRUE` for the user's own row and every descendant row, giving the manager their whole subtree. Recipe 4 (a precomputed bridge) usually scales better on large hierarchies.

---

## Recipe 6: Partial RLS — "% of all" without leaking other rows

A measure cannot override RLS (it cannot even detect RLS). To show a ratio of the user's region over **all** regions, query a **summary table that has no RLS filter**. Build a hidden calculated summary table and apply RLS only to the `Salesperson` table.

**Hidden summary calculated table** (no RLS rule applied to it):
```dax
SalesRevenueSummary =
SUMMARIZECOLUMNS(
    Sales[OrderDate],
    "RevenueAllRegion", SUM(Sales[Revenue])
)
```

**RLS rule on `Salesperson` only:**
```dax
[EmailAddress] = USERPRINCIPALNAME()
```

**Measure (% of all regions):**
```dax
Revenue % All Region =
DIVIDE(
    SUM(Sales[Revenue]),
    SUM(SalesRevenueSummary[RevenueAllRegion])
)
```

`Sales` is filtered by the user's region (numerator); `SalesRevenueSummary` is related only to `Date`, not to `Salesperson`, so it stays unfiltered (denominator = all regions). An **aggregation table** achieves the same. Caution: if only two regions exist, a user can infer the other region's value — don't expose sensitive totals this way.

---

## Recipe 7: Embedding with CUSTOMDATA()

In "embed for your customers," the app passes a value via the connection string, read by `CUSTOMDATA()`. Treat it like Recipe 3 — validate explicitly.

**Rule:**
```dax
[TenantId] = CUSTOMDATA()
```

Because app logic can pass *any* value, ensure accidental/malicious values yield no rows (combine with an `IF`/`FALSE()` fallback when branching on it).

---

## Recipe 8: Diagnostic "Who Am I" measure

Not a rule — a measure to publish on a Card to confirm which identity the service passes (fastest way to debug "no data").

```dax
Who Am I = USERPRINCIPALNAME()
```

If the displayed UPN doesn't match a row in the security table, that's why the user sees nothing — fix the stored UPN or group membership.
