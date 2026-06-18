# Static vs Dynamic RLS — Decision Guide

## Contents
- Step 0: Should you use RLS at all?
- Decision matrix (avoid / static / dynamic)
- Static RLS in depth
- Dynamic RLS in depth
- Choosing the identity function (USERPRINCIPALNAME vs USERNAME vs CUSTOMDATA)
- Worked decisions

## Step 0: Should you use RLS at all?

RLS is not always the right tool. If you have only a few simplistic static filters and the audiences **never overlap**, consider publishing **separate semantic models** instead of one model with roles.

**Separate-models pattern:** publish the *same* model multiple times, each filtered by a different **query parameter** value (cross-ref `transforming-data-power-query`), into one **workspace (or app) per audience**. No roles are defined. Each audience is granted access to only its workspace/app.

| | RLS (one model + roles) | Separate models (no RLS) |
|---|---|---|
| Query performance | Filters added to every query (slower) | Fewer filters (faster) |
| Model size | One larger model | Several smaller models (easier under capacity limits) |
| Maintenance | One model, one report set | Duplicated reports/dashboards per audience |
| Features that conflict with RLS (e.g. Publish to web) | Blocked | Available |
| High-privilege user wanting a consolidated view | Single report | Must open multiple reports |

**Rule of thumb:** few audiences, static non-overlapping filters, and a desire to use RLS-incompatible features → separate models. Many audiences, overlapping/consolidated views, sensitive data needing a defense-in-depth layer → RLS.

## Decision matrix

```
Need per-user data restriction?
├─ No  ............................. no roles (everyone sees all rows)
└─ Yes
   ├─ Few, fixed, non-overlapping audiences AND want RLS-incompatible features?
   │     -> AVOID RLS: publish separate models (query-parameter pattern)
   ├─ Small fixed set of audiences, filters are constants, set rarely changes?
   │     -> STATIC RLS: one role per audience, constant DAX filters
   └─ Many users / users change often / a user maps to "their own" rows?
         -> DYNAMIC RLS: store identities in the model, one role filters by current user
            └─ User must also see everyone *below them* in a hierarchy?
                  -> DYNAMIC RLS + bridge/security table (see dynamic-rls-dax-recipes.md)
```

## Static RLS in depth

- **One role per audience.** Each role's rule(s) use **constant** DAX, e.g. `[Region] = "Midwest"`.
- **Mapping:** assign the matching security group to each role in the service after publishing.
- **Pros:** simplest to reason about; no identity table to maintain.
- **Cons:** every new audience = a new role + new mapping; does not scale to many or fast-changing audiences.
- **Watch out:** a user mapped to two static roles sees the **union** (filters are additive). Keep mappings non-overlapping, or build a single role that covers the combined need.

```dax
-- Role "Midwest"
[Region] = "Midwest"

-- Role "All Regions" (super-user): a role with NO rules sees all rows
-- (define the role, add no table rules)
```

## Dynamic RLS in depth

- **One role** that filters by the **current user's identity** returned at query time.
- Requires a table in the model that stores the user identities (UPNs) and the attribute that drives the filter (region, department, etc.). Keep this table refreshed from Microsoft Entra ID — a common approach is a **dataflow** kept in sync with Entra so creators can build a data-driven model (cross-ref `connecting-data-sources`, `transforming-data-power-query`).
- The identity table is usually **hidden** and related to the fact table so the filter propagates.
- **Pros:** one role serves the whole organization; new users need no model change, only a new row + group membership.
- **Cons:** depends on accurate, current UPN data; a missing/mistyped UPN means that user sees **no data**.

```dax
-- Single dynamic role "Salesperson", rule on the hidden Salesperson table
[EmailAddress] = USERPRINCIPALNAME()
```

The matched salesperson's `Region` then propagates to `Sales` through the relationship.

## Choosing the identity function

| Function | Returns | Use when |
|---|---|---|
| `USERPRINCIPALNAME()` | The user's UPN (email-style) | **Default choice.** Match against stored UPN/email column; stable across Desktop and the service |
| `USERNAME()` | The user's name | Legacy/AD scenarios; note the value's **format can differ** between Desktop preview and the service — test both |
| `CUSTOMDATA()` | A value passed in the connection string | **Embedding** ("embed for your customers"); the app passes an effective identity |

**Security warning for embedding / `USERNAME`:** when an app can pass any value as the effective identity, never write a rule whose *fallback* grants all rows. Test every expected value explicitly; any unexpected value must return **no** rows (see the IF-per-value pattern in dynamic-rls-dax-recipes.md and the View As checklist).

## Worked decisions

1. **"Two sales regions, separate teams, we also want Publish to web."**
   → AVOID RLS. Publish two parameterized models to two workspaces. RLS would block Publish to web.

2. **"Five fixed departments, filters are just `[Dept] = "..."`, rarely changes."**
   → STATIC RLS. Five roles, one per department, map a security group to each.

3. **"3,000 salespeople, each sees only their own customers, people join/leave weekly."**
   → DYNAMIC RLS. Hidden Salesperson table with UPNs (refreshed via dataflow), one role: `[EmailAddress] = USERPRINCIPALNAME()`.

4. **"A regional manager must see their own sales plus all reps reporting to them."**
   → DYNAMIC RLS + bridge/security table over the org hierarchy (PATH-based or path table). See dynamic-rls-dax-recipes.md.

5. **"Embedded portal where our app authenticates external customers."**
   → DYNAMIC RLS with `CUSTOMDATA()` (or a passed effective identity); harden against unexpected values.
