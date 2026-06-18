# Role Testing (View As) Checklist

Test **every** role before publishing, and re-test after any model or relationship change. RLS bugs are silent: a wrong rule shows plausible-but-wrong data, not an error.

## Where: View As

In **Power BI Desktop**, on the **Modeling** ribbon tab, use **View As** to render the report as a chosen role (and, for dynamic roles, as a specific user via **Other user**). This applies the role's filters to all visuals so you can verify what that role sees.

> Paginated reports note: the View As / Test as role feature does **not** work for paginated reports. Paginated RLS is set up differently (parameters bound to the built-in **UserID** field in Power BI Report Builder, used in a filter or query); validate those by previewing/running with real identities.

## Procedure

1. Add the diagnostic measure `Who Am I = USERPRINCIPALNAME()` to a Card (remove before release if desired).
2. For each **static** role: View As that role; confirm only the intended rows appear; confirm the grand total matches the expected subset.
3. For each **dynamic** role: View As the role **and** supply **Other user** values:
   - an **expected** user → correct subset of rows.
   - an **unexpected / malformed** value (e.g. a typo'd UPN) → **no rows** (never all rows).
4. Verify **multi-role** users: if a person can map to two roles, confirm the **additive union** is acceptable (or redesign to one role).
5. Confirm **filter propagation**: a filter on the security/dimension table reaches the fact table through an **active** relationship. Where the filter must travel against the relationship direction, confirm **Apply security filter in both directions** is enabled.
6. Measure performance impact: capture visual query durations with **Performance Analyzer** without RLS, then with View As, and compare (RLS adds filters to every query).
7. After publishing: map **security groups** (not individuals) to roles in the service; only the model owner / workspace admin can do this.
8. Confirm **permission level**: consumers/creators who must stay under RLS have **Read** only (Write/editor roles bypass RLS).

## Copy-paste checklist

```
RLS role testing:
- [ ] Who Am I (USERPRINCIPALNAME) card added for diagnostics
- [ ] Every STATIC role viewed via View As -> correct rows + correct totals
- [ ] Every DYNAMIC role tested with an EXPECTED user -> correct rows
- [ ] Every DYNAMIC role tested with an UNEXPECTED/typo value -> NO rows
- [ ] Hierarchy roles: manager sees own rows + all subordinates (and nothing above/sideways)
- [ ] No user unintentionally maps to multiple roles (additive union checked)
- [ ] Filters propagate through ACTIVE relationships to the fact table
- [ ] "Apply security filter in both directions" enabled only where required, and re-tested
- [ ] Tables contain data and correct values (empty/wrong data mimics RLS bugs)
- [ ] Aggregation/summary tables (if any) filter consistently with detail tables
- [ ] Performance Analyzer: query durations acceptable with RLS enforced
- [ ] After publish: security GROUPS mapped to roles (owner/admin only)
- [ ] Consumers/creators under RLS have READ permission only (not Write)
- [ ] Diagnostic measure removed or hidden before release (optional)
```

## Troubleshooting matrix

| Symptom | Likely cause | Fix |
|---|---|---|
| User sees **all** data | Multiple roles (union), a `TRUE()` fallback, or Write permission | One role; explicit per-value rules; downgrade to Read |
| User sees **no** data | UPN missing/mistyped in security table; user not mapped to any role | Use `Who Am I` to compare; fix UPN / group membership |
| Partial/missing rows across a relationship | Filter can't propagate the needed direction; inactive relationship | Enable bidirectional security filter; use an active relationship |
| Report looks "broken" (field error) | Visual references an RLS-restricted reference, or an OLS-restricted object | Per-audience report design; check OLS scope (Tabular Editor) |
| Inconsistent results with aggregations | RLS rules don't filter aggregation and detail consistently | Apply matching RLS to aggregation tables |
| Slow visuals only when RLS on | RLS filters every query; weak model design | Filter dimensions not facts; optimize DirectQuery source |
