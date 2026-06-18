---
name: securing-row-level-access
description: Restricts which data each user sees in a Power BI semantic model using row-level security (RLS) roles and DAX filter rules, choosing between static and dynamic RLS, building bridge/security tables for organizational hierarchies, applying object-level security (OLS) for sensitive tables/columns, testing roles with View As, and classifying content with Microsoft Purview sensitivity labels. Use when implementing RLS, dynamic RLS with USERPRINCIPALNAME or USERNAME, hierarchical or manager/employee security, object-level security, sensitivity labels or information protection, or when a user sees too much or no data, a broken-looking report, or RLS behaves unexpectedly with bidirectional relationships.
---

# Securing Row-Level Access in Power BI

## Purpose

RLS filters **table rows** so a consumer sees only the data they are entitled to (for example, only their own sales region) from a single shared report. RLS does **not** restrict access to model *objects* — tables, columns, or measures — that is object-level security (OLS). Sensitivity labels classify and (optionally) encrypt content but do not, by themselves, change who can open it in the service. This skill covers all three plus role testing.

**Prerequisites (from the full-lifecycle plan):** Depends on `modeling-semantic-data` (relationships, cardinality, cross-filter direction), `authoring-dax-measures` (filter context, DAX expressions), and `publishing-and-sharing-content` (workspace roles, apps). Cross-references the Kimball skill `conformed-dimension-management` (a security/bridge table is a dimension; org hierarchies use the same surrogate-key and shrunken-dimension thinking).

## When to Use

- Restricting rows per user/region/department in a shared semantic model.
- Deciding **static vs dynamic** RLS, or whether to avoid RLS entirely (publish separate models).
- Implementing **dynamic RLS** with `USERPRINCIPALNAME()`, `USERNAME()`, or `CUSTOMDATA()`.
- Securing an **organizational hierarchy** (a manager sees their whole team) via a bridge/security table.
- Hiding sensitive **tables/columns** (salary) — use OLS, not RLS.
- Applying **sensitivity labels** / Microsoft Purview information protection.
- Diagnosing: a user sees all data, sees no data, or a visual looks "broken."

## Core decisions (in order)

1. **Do you even need RLS?** A few simplistic static filters with non-overlapping audiences → often better to publish **separate models** (different query parameter values, one workspace/app per audience) than to build RLS. See [static-vs-dynamic-rls-guide.md](static-vs-dynamic-rls-guide.md) for the full trade-off.
2. **Static or dynamic?** Static = constant filters (`[Region] = "Midwest"`), one role per audience. Dynamic = the model stores user identities and one role filters by the current user. Use dynamic when audiences are many or change often. See the decision guide.
3. **Flat or hierarchical?** If a user must see their own rows *plus* everyone reporting to them, you need a bridge/security table over the org hierarchy — see [dynamic-rls-dax-recipes.md](dynamic-rls-dax-recipes.md).
4. **Rows or objects?** Hiding columns/tables (not rows) = OLS, set in Tabular Editor (no Power BI Desktop UI). See the OLS section below.

## How RLS works (two parts)

RLS has exactly two pieces:

- **Rules** — defined in Power BI Desktop. Create one or more **roles**; each role applies DAX filter expressions to model tables. A rule is evaluated in **row context**: when the expression returns `TRUE` for a row, that row is visible. By default a model has **no roles**, which means everyone who can query the model sees **all** rows.
- **Role mappings** — done **after publishing** to the service. Only the **semantic model owner or workspace admin** can map members to roles. Map **security groups** (managed in Microsoft Entra ID) to roles whenever possible, not individual users.

### Static rules

DAX referring to constants. One role per fixed audience.

```dax
-- Role "Midwest": only Midwest rows
[Region] = "Midwest"
```

### Dynamic rules

DAX using environment functions that return the current user, so a **single role** serves everyone:

- `USERPRINCIPALNAME()` — returns the user's UPN (email-style). **Preferred** for matching against stored UPNs.
- `USERNAME()` — returns the user's name; format can differ between Desktop preview and the service.
- `CUSTOMDATA()` — a value passed via the connection string (embedding scenarios).

```dax
-- Role "Salesperson" on a hidden Salesperson dimension
[EmailAddress] = USERPRINCIPALNAME()
```

The matched user's row in the security/dimension table propagates its filter (for example, `Region`) to the fact table **through model relationships**. Full recipes (UPN match, manager hierarchy via bridge table, partial RLS for "% of all") are in [dynamic-rls-dax-recipes.md](dynamic-rls-dax-recipes.md).

## Design rules that keep RLS correct and fast

- **Filter dimensions, not facts.** Enforce RLS on small dimension tables and let well-designed relationships propagate the filter to the fact table. RLS adds a filter to *every* DAX query, so model design drives performance. Follow star-schema guidance (cross-ref `modeling-semantic-data`, Kimball `conformed-dimension-management`).
- **RLS propagates only through *active* relationships.** Avoid `LOOKUPVALUE()` to achieve what a relationship can do.
- **Single role per user.** If a user maps to **multiple roles**, the filters are **additive** (union) — they see the *union* of all rows. There is **no "once denied, always denied"**: a `FALSE()` (deny-all) role plus a `TRUE()` (allow-all) role means the user sees **everything**. Design one role that grants exactly the needed permissions.
- **A role with no rules sees all rows** — intentionally useful for an admin/super-user role.
- **DirectQuery:** when RLS filters DirectQuery tables with relationships to other DirectQuery tables, optimize the **source** (indexes, persisted computed columns). Honoring source RLS via SSO is an alternative (cross-ref `connecting-data-sources`).

## RLS and bidirectional filters (cross-ref `modeling-semantic-data`)

RLS filters propagate only in the relationship's cross-filter direction. When a role's filter must travel against the normal single direction (common in many-to-many / bridge designs), you must enable **Apply security filter in both directions** on the relationship. If RLS gives unexpected results, this property being unset is a top cause. Bidirectional cross-filtering is powerful but can over-share or hurt performance — enable it deliberately, only where the security model requires it, and re-test with View As.

## Object-level security (OLS) — for columns/tables, not rows

Use OLS to hide **sensitive columns or tables** (e.g., salary) so that, for unauthorized users, the object appears not to exist. Notes:

- Any **measure that references a restricted column is itself restricted**.
- A visual that includes a restricted object shows an **error** ("object doesn't exist") to unauthorized users — looks broken; design reports per audience.
- **There is no Power BI Desktop UI for OLS.** Configure it with **Tabular Editor** (third-party external tool).
- **Perspectives are not security** — a user can still query objects outside a perspective; treat perspectives as a convenience only.

## RLS is enforced only with Read-level permission

RLS is bypassed for editors. The permission rules:

| Permission on the semantic model | RLS enforced? |
|---|---|
| Read | Yes |
| Read + Build | Yes |
| Write (Contributor/Member/Admin workspace role, or per-item Write) | **No** — sees all data |

Therefore, when RLS must hold, consumers and report creators must have **only Read** (commonly via a **Power BI app** or the **Viewer** role). For self-service report creators who must stay under RLS, put the RLS model in a **separate data workspace** where they have only Read + Build, and let them author in a separate reporting workspace (cross-ref `publishing-and-sharing-content`).

## Sensitivity labels / information protection (Microsoft Purview)

Sensitivity labels classify content and can apply encryption to files. Key facts:

- Labels are a **prerequisite-heavy, org-wide** Microsoft Purview effort (labels and label policies are created in the Microsoft Purview compliance portal; Power BI behavior is then enabled via **Fabric tenant settings**).
- **A label does not change who can open content in the Power BI service** — standard permissions (workspace roles, app, sharing) still govern access. Labels matter most when content is **exported/encrypted**: a label (with encryption) follows exports to Excel, PowerPoint, PDF, and Power BI Desktop files, so only authorized RMS users can open them.
- One label per item; apply the **most restrictive** label when data spans multiple classifications.
- Labels can be **inherited** from upstream data sources and flow to **downstream** items; both via tenant settings.
- Use **mandatory** and/or **default** label policies to ensure content is always labeled.

Full label structure, encryption, inheritance, and tenant-setting checklist: [info-protection-and-labels.md](info-protection-and-labels.md).

## Validate every role (View As) — do this before publishing

Use **View As** on the **Modeling** ribbon tab in Power BI Desktop to test each role. Always test, and for dynamic rules test **expected *and* unexpected** values (an accidental or malicious value should return **no** rows, never all rows). Copy the checklist in [role-testing-view-as-checklist.md](role-testing-view-as-checklist.md) and check off every role.

> Tip: add a diagnostic measure `Who Am I = USERPRINCIPALNAME()` on a Card and publish it; it reveals exactly which identity the service passes, which quickly explains a "no data" report caused by a missing/mistyped UPN.

## Common pitfalls and corrections

| Pitfall | Symptom | Correction |
|---|---|---|
| User mapped to multiple roles | Sees more than intended (additive union) | One role per user; map security groups, not overlapping roles |
| Deny role + allow role on same user | Deny is ignored; user sees all | No "once denied always denied" — consolidate into one role |
| Dynamic rule with a fallback `TRUE()` for "all other values" | Typo/unexpected UPN returns ALL rows | Test each expected value explicitly; unexpected → `FALSE()` (no rows) |
| Editor permission (Write) on the model | RLS not enforced for that user | Give consumers/creators **Read** only; use an app or Viewer role |
| RLS filter must cross a relationship the wrong way | User sees no data / partial data | Enable **Apply security filter in both directions**; verify active relationships |
| `LOOKUPVALUE()` used for security instead of a relationship | RLS doesn't propagate | Replace with a proper active relationship |
| Missing/mistyped UPN in the security table | Specific user sees no data | Verify UPN values (keep them refreshed from Entra/dataflow); use the `Who Am I` measure |
| OLS attempted in Power BI Desktop | No UI to do it | Configure OLS in **Tabular Editor** |
| Sensitivity label assumed to block service access | Users still open content | Labels classify/encrypt exports; use RLS/OLS/permissions for access control |

## Success criteria

- Each role tested with **View As**; expected values show correct rows, unexpected values show **none**.
- Consumers/creators who must stay under RLS hold **only Read** permission.
- RLS rules sit on **dimensions**, propagate through **active** relationships, and bidirectional security is enabled only where required.
- Each user maps to **one** role (preferably via a security group), avoiding additive surprises.
- Dynamic rules match on `USERPRINCIPALNAME()` (or documented `USERNAME`/`CUSTOMDATA`).
- Sensitive columns/tables handled with OLS; sensitivity labels applied per the org's label policy.

## Reference files

- [static-vs-dynamic-rls-guide.md](static-vs-dynamic-rls-guide.md) — decision guide: avoid RLS vs static vs dynamic, with trade-offs.
- [dynamic-rls-dax-recipes.md](dynamic-rls-dax-recipes.md) — copy-paste DAX: UPN match, manager/org hierarchy via bridge table, partial "% of all" pattern, embedding/CUSTOMDATA.
- [role-testing-view-as-checklist.md](role-testing-view-as-checklist.md) — View As test procedure and per-role checklist.
- [info-protection-and-labels.md](info-protection-and-labels.md) — sensitivity labels, encryption, inheritance, tenant settings.

---
*Provenance: built per SKILL 18 "Securing Row-Level Access" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded in the Power BI docs `guidance/rls-guidance.md`, `guidance/powerbi-implementation-planning-security-overview.md`, `-security-content-creator-planning.md`, `-security-report-consumer-planning.md`, `guidance/powerbi-implementation-planning-info-protection.md`, and `paginated-reports/paginated-reports-rls.md`; cross-references Kimball `conformed-dimension-management`.*
