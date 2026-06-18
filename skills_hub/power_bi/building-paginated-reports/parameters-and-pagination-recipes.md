# Parameters and Pagination Recipes

Concrete recipes for report parameters (including cascading), pagination control, and row-level security in Power BI Report Builder. All query examples use Transact-SQL against an Azure SQL Database with a `Reseller` table and a related `Sales` table.

## Contents
- Report parameters: how they are created
- Cascading parameters: core procedure
- Cascading recipe A — filter by related columns (geography drill)
- Cascading recipe B — filter by grouping column (first letter / letter bands)
- Cascading recipe C — filter by search pattern
- Cascading recipe D — present only relevant items (fact-driven)
- Pagination: repeat headings, page breaks, page names
- Page layout: physical page, margins, newsletter columns
- Document map (table of contents)
- Row-level security with the UserID built-in field
- Performance notes

## Report parameters: how they are created

Report parameters are created three ways:
- **Automatically** when a dataset query contains **query variables** (each `@var` creates a matching dataset query parameter and report parameter of the same name; a query parameter may reference a query variable or a stored-procedure input parameter).
- **Automatically** when you add a reference to a shared dataset that has query parameters.
- **Manually** in the **Report Data** pane.

Parameters filter report data (before/after data is returned), control appearance, pass values to subreports, or support what-if. When you publish, parameters are saved and managed separately from the definition. **Order matters** for cascading parameters — the order in the Report Data pane sets the order users choose values, and a later parameter's query references earlier parameters.

## Cascading parameters: core procedure

Cascading parameters are ordered parameters where each parameter's **available values** depend on an earlier selection (e.g. Category then Subcategory). Build them like this:

1. Create the **main dataset** whose query has one **query parameter** per cascading parameter (typically in the `WHERE` clause). Run it once (the **!** button) — report parameters matching the query parameters are auto-created in that order.
2. Create a **separate dataset per parameter** to supply that parameter's available values:
   - The **independent** parameter's value query usually has **no** query parameters, e.g. `SELECT DISTINCT <column> FROM <table>`.
   - Each **dependent** parameter's value query includes a query parameter for each parameter it depends on.
3. For each parameter: **Parameter Properties > Available Values > Get values from a query**, pick the dataset, and set the **Value** field and **Label** field.
4. Map each non-independent dataset's query parameters to the corresponding report parameters.
5. **Run** to test: choosing the independent value runs the next parameter's query; the report does **not** run automatically after the last parameter, so select **View Report**.

## Cascading recipe A — filter by related columns (geography drill)

Five parameters (Country-Region > State-Province > City > PostalCode > Reseller). Each value dataset filters by all earlier selections.

```sql
-- CountryRegion (independent)
SELECT DISTINCT [Country-Region]
FROM [Reseller]
ORDER BY [Country-Region]

-- StateProvince (depends on @CountryRegion)
SELECT DISTINCT [State-Province]
FROM [Reseller]
WHERE [Country-Region] = @CountryRegion
ORDER BY [State-Province]

-- City (depends on @CountryRegion, @StateProvince)
SELECT DISTINCT [City]
FROM [Reseller]
WHERE [Country-Region] = @CountryRegion
  AND [State-Province] = @StateProvince
ORDER BY [City]

-- Reseller (final; depends on all four)
SELECT [ResellerCode], [ResellerName]
FROM [Reseller]
WHERE [Country-Region] = @CountryRegion
  AND [State-Province] = @StateProvince
  AND [City] = @City
  AND [PostalCode] = @PostalCode
ORDER BY [ResellerName]
```

Continue the same pattern for the **PostalCode** dataset. Each `@parameter` may live in a `SELECT` or be passed to a stored procedure.

## Cascading recipe B — filter by grouping column (first letter / letter bands)

User picks the first letter of the reseller name; the second parameter lists matching resellers.

```sql
-- ReportGroup (independent): distinct first letters
SELECT DISTINCT LEFT([ResellerName], 1) AS [ReportGroup]
FROM [Reseller]
ORDER BY [ReportGroup]

-- Reseller (depends on @ReportGroup)
SELECT [ResellerCode], [ResellerName]
FROM [Reseller]
WHERE LEFT([ResellerName], 1) = @ReportGroup
ORDER BY [ResellerName]
```

For best performance, materialize the grouping column on the table (persisted + indexed) instead of computing it at query time:

```sql
ALTER TABLE [Reseller]
ADD [ReportGroup] AS LEFT([ResellerName], 1) PERSISTED
```

To group by **letter bands** plus a supporting index:

```sql
ALTER TABLE [Reseller]
ADD [ReportGroup2] AS CASE
  WHEN [ResellerName] LIKE '[A-C]%' THEN 'A-C'
  WHEN [ResellerName] LIKE '[D-H]%' THEN 'D-H'
  WHEN [ResellerName] LIKE '[I-M]%' THEN 'I-M'
  WHEN [ResellerName] LIKE '[N-S]%' THEN 'N-S'
  WHEN [ResellerName] LIKE '[T-Z]%' THEN 'T-Z'
  ELSE '[Other]'
END PERSISTED
GO

CREATE NONCLUSTERED INDEX [Reseller_ReportGroup2]
ON [Reseller] ([ReportGroup2]) INCLUDE ([ResellerCode], [ResellerName])
GO
```

## Cascading recipe C — filter by search pattern

User types a search term; the second parameter lists resellers whose name contains it.

```sql
-- Reseller (depends on @Search)
SELECT [ResellerCode], [ResellerName]
FROM [Reseller]
WHERE [ResellerName] LIKE '%' + @Search + '%'
ORDER BY [ResellerName]
```

To let users supply their own wildcard, use `WHERE [ResellerName] LIKE @Search` (so `red%` matches names starting with "red"). Because many users expect `*` rather than `%`, you can translate it:

```sql
WHERE [ResellerName] LIKE SUBSTITUTE(@Search, '%', '*')
```

## Cascading recipe D — present only relevant items (fact-driven)

Use fact data to offer only items with recorded activity. Two date parameters bound a range; the reseller parameter lists only resellers with orders in that range.

```sql
-- Reseller (depends on @OrderDateStart, @OrderDateEnd)
SELECT DISTINCT [r].[ResellerCode], [r].[ResellerName]
FROM [Reseller] AS [r]
INNER JOIN [Sales] AS [s]
  ON [s].[ResellerCode] = [r].[ResellerCode]
WHERE [s].[OrderDate] >= @OrderDateStart
  AND [s].[OrderDate] < DATEADD(DAY, 1, @OrderDateEnd)
ORDER BY [r].[ResellerName]
```

The `< DATEADD(DAY, 1, @OrderDateEnd)` pattern includes the whole end day regardless of time component.

## Pagination: repeat headings, page breaks, page names

**Repeat column headings on every printed page** (so a long tablix stays readable):
1. In the **Grouping** pane, open the top-bar drop-down > **Advanced Mode**.
2. Select the top **Static** bar under **Row Groups** (this selects the heading cell).
3. In **Properties** (for **Tablix Member**) set **KeepWithGroup = After** and **RepeatOnNewPage = True**.

**Page breaks** are supported on table/matrix/list data regions, **groups**, and **rectangles**:
- **BreakLocation** — Start, End, StartAndEnd (Between for groups). Set in the Tablix/Rectangle/Group **Properties** dialog; **cannot** be set with an expression.
- **Disabled** — True ignores the break; can be an expression to disable breaks dynamically at run time.
- **ResetPageNumber** — True resets the page number to 1 at the break; can be an expression.

Set **Disabled**, **ResetPageNumber**, and **PageName** in the **Properties** pane (under the **PageBreak** category; for groups it is inside the **Group** category).

**Page names:**
- **InitialPageName** (report property) is the default page name; used for all pages if no per-break name is set.
- **PageName** on a table/matrix/list/group/rectangle names the new page a break creates. If blank, InitialPageName is used; if that is also blank, the page is unnamed.
- On **Excel** export, each page defined by a break becomes a **worksheet** named by **PageName** (or InitialPageName). Reference current page name/number in expressions via the **Globals** collection.

## Page layout: physical page, margins, newsletter columns

- The **report body** is a container that grows/shrinks to fit content and can span pages; it does **not** equal the physical page.
- The **physical page** (paper) defaults to **8.5 × 11 inches**. Set `PageHeight`/`PageWidth`, margins (`TopMargin`/`BottomMargin`/`LeftMargin`/`RightMargin`), and columns in the **Report Properties** pane, the **Properties** pane, or the **Page Setup** dialog (available after **Run**).
- **Hard-page-break** renderers (PDF, Image) insert breaks horizontally and vertically by physical size; **soft-page-break** renderers (Excel, Word, HTML, MHTML) add breaks as needed. Margins apply only to hard-page-break formats.
- If a one-page-wide design renders across multiple pages, body **width + margins** exceeds the page width — drag the body container corner left to shrink it.
- **Newsletter-style columns** divide a physical page into vertical logical columns (left-to-right, top-to-bottom); they apply only to **PDF or Image** and are **not** supported in subreports.

## Document map (table of contents)

A document map is a side pane of links (like a table of contents) that jumps to report items. Set the **DocumentMapLabel** property of a report item (text or expression). For group values, open **Edit Group > Advanced > Document map** and set an expression that matches the group expression. **Note:** document maps **do not render in the Power BI service** — they render on **export** (PDF Bookmarks pane, Word table of contents, Excel worksheet of links; ignored by Atom/TIFF/XML/CSV).

## Row-level security with the UserID built-in field

Paginated reports run on the **SQL Server Reporting Services engine**, so you set RLS filtering in Report Builder (not the Analysis Services engine). Prerequisite for a semantic-model source: at least a **Viewer** role in the dataset workspace and **Build** permission on the dataset; RLS works only with **SSO** authentication.

1. Create a report **parameter** and assign it the built-in field **UserID** (the authenticated user).
2. Use UserID either in a **dataset filter** or in the **query**:

**Filter approach** — in **Dataset Properties > Filter**, set the **Expression** to your parameter, the **Value** (via the function button) to **Built-in Fields > UserID**, so the filter reads `<your parameter> = UserID`.

**Query approach** — in **Dataset Properties > Parameters**, **Add** a parameter named `@UserID` with value `[&UserID]`, then reference it in the query, e.g.:

```sql
SELECT *
FROM FinalTable
WHERE FinalTable.Color = @UserID
```

(Replace the column/logic with your actual security column; you can build more complex queries.)

**Limitation:** the **Test as role / View as role** feature does **not** work for paginated reports. Verify by signing in as the actual user. Note that the UserID value can differ between Report Builder preview and the service depending on whether the machine uses Active Directory or Microsoft Entra ID. (Relates to skill `securing-row-level-access`.)

## Performance notes

- Prefer **stored procedures** for parameter and main queries — cached query plans run faster and allow richer logic. **Exception:** stored procedures are **not supported** for gateway relational sources (SQL Server, Oracle, Teradata) — use parameterized text queries there.
- Add **indexes** to support parameter value queries so drop-downs populate quickly and the database is not overburdened.
- **Materialize** computed grouping columns (PERSISTED + indexed) instead of computing them at query time.
- Cascading parameters are inherently efficient because each retrieves a **smaller** set of available values.
