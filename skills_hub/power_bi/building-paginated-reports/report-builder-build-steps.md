# Power BI Report Builder Build Steps

End-to-end walkthrough: connect, build a tablix, format for print, and publish. Steps mirror the Adventure Works tutorial (Azure SQL Database source, Table or Matrix Wizard).

## Contents
- Prerequisites and licensing
- Step 1: Create the dataset and embedded data source
- Step 2: Build the matrix with the wizard
- Step 3: Inspect what the wizard created (tablix, groups)
- Step 4: Run the report
- Step 5: Format for print
- Step 6: Publish to the Power BI service
- Supported data sources (reference table)

## Prerequisites and licensing

- Install **Power BI Report Builder** from the Microsoft Download Center (`https://aka.ms/pbireportbuilder`) — free.
- Licensing matches `.pbix`: a **free** license publishes paginated reports only to **My Workspace**; a **Pro** or **Premium Per User (PPU)** license publishes to other workspaces and requires at least a **Contributor** role in that workspace.

## Step 1: Create the dataset and embedded data source

1. Start Power BI Report Builder. In the **Getting Started** dialog, with **New Report** selected, choose **Table or Matrix Wizard**.
2. On **Choose a dataset**, select **Create a dataset** > **Next**.
3. On **Choose a connection to a data source**, select **New**. The **Data Source Properties** dialog opens.
4. Name the data source (characters and underscores only), e.g. `MyAzureDataSource`.
5. In **Select connection type**, choose the source type (e.g. **Microsoft Azure SQL Database**).
6. Select **Build** next to **Connection string**; in **Connection Properties** enter the **Server name**, choose authentication (e.g. **Use SQL Server Authentication** with user/password), pick the database, then **Test Connection** > **OK** > **OK**.
7. Back on **Choose a connection to a data source**, the new source shows "(in this Report)". Select it > **Next**.
8. On **Design a query**, expand the schema/tables, select the tables you need (relationships are auto-detected when **Relationships > Auto Detect** is on), select **Run Query** to see **Query results**, then **Next**.

The result is an **embedded data source** plus an **embedded dataset** in the **Report Data** pane. (You can also create a dataset manually: right-click a data source > **Add Dataset**, choose **Query type** = **Text**, and type the query.)

## Step 2: Build the matrix with the wizard

1. On **Arrange fields**, drag dimension fields to **Row groups** (e.g. CompanyName, SalesOrderNumber, Product_Name) and numeric fields to **Values** (e.g. OrderQty, UnitPrice, LineTotal). Values become **sums** automatically.
2. **Next**. On **Choose the layout**, clear **Expand/collapse groups** when you want the table to wrap to multiple pages (otherwise it stays collapsed/interactive).
3. **Next** > **Finish**. The tablix appears on the design surface.

## Step 3: Inspect what the wizard created (tablix, groups)

- The **Report Data** pane shows the embedded data source and dataset.
- The data region is a **tablix** named `Tablix1` (a tablix displays data in rows and columns; it can be a table or a matrix).
- The **Grouping** pane shows the **Row Groups** (and **Column Groups**, if any) you defined.
- If the **Properties** pane is hidden: **View** tab > **Properties**.

## Step 4: Run the report

- Select **Run** on the **Home** ribbon. Report Builder retrieves only enough rows for a few pages at a time, so the page count may first show `1 of 2?` (the `?` means it's still loading). The full table has many more rows than design view showed.

## Step 5: Format for print

1. **Page width / margins.** Drag the ruler to set the body width (e.g. 7 in for an 8.5-in page). Select the gray area outside the body to show **Report** properties; expand **Margins** and set **Left**/**Right** (e.g. from 1 in to 0.75 in).
2. **Title.** Select **Click to add title**, type the title; set **Font > Color** in **Properties**.
3. **Page numbers (footer).** Expand **Built-in Fields** in the Report Data pane. Drag **Page Number** to the footer; add a **Text Box** (Insert tab) with "of"; drag **Overall Total Pages** after it. The footer already has **[&ExecutionTime]** (date/time stamp).
4. **Column widths.** Select the tablix and a column header; drag the right edge to widen.
5. **Currency.** Select the value cells (click first, Shift-click last). On **Home**, select the **$** currency button, then the arrow next to **Placeholder styles > Sample Values** to preview formatted values.
6. **Repeat headings on every page.** See `parameters-and-pagination-recipes.md` (RepeatOnNewPage). Use **Run > Print Layout** to verify the printed pagination.
7. **Save** the `.rdl` to your computer.

## Step 6: Publish to the Power BI service

**From Report Builder:**
1. **File > Publish** (if not signed in, select **Sign in** top-right).
2. Select a workspace, type a **File name** > **Save**.
3. Open `https://app.powerbi.com`, find the workspace, open the report. If it has **parameters**, you must select them before viewing.

**From the service (upload):**
1. Save the `.rdl` locally; in the target workspace select **Upload > Browse** and pick the file. (You can't import from OneDrive/SharePoint.)

**Set credentials / gateway:**
- Select the report's **... > Manage**, then **Continue > Edit credentials**, configure, **Sign in**.
- For **on-premises** sources, install/bind an **on-premises data gateway** via the settings icon > **Manage connections and gateways**; for paginated reports with a gateway, the publisher needs gateway **admin** permission (and must specify which data-source connection to manage).

## Supported data sources (reference table)

Paginated reports have **no data model** — sources are embedded. (Relates to skill `connecting-data-sources`.)

**Natively supported (ready after upload; default to SSO where applicable):**

| Data source | Authentication | Notes |
|-------------|----------------|-------|
| Azure SQL Database / Azure Synapse Analytics | Basic, SSO, OAuth2 | Can also use an Enterprise gateway. For first use in a workspace, set an auth type before running. |
| Azure SQL Managed Instance | Basic, SSO, OAuth2 | VNet-local endpoints route through Enterprise gateway. |
| Azure Analysis Services | SSO, OAuth2 | Can't use Enterprise/VNet gateway; service principal not supported with SSO. |
| Power BI semantic model | SSO | Import or DirectQuery models; needs Read permission. DirectQuery source queries have a fixed **10-minute** timeout — for longer, use the model's XMLA Read/Write endpoint. |
| Direct Lake models | SSO, OAuth2 | Direct Lake on SQL endpoints: SSO + OAuth2; on semantic models: SSO. |
| Premium semantic model (XMLA) | SSO | Set **Do not use credentials**; honors workspace/app security-group membership. **RLS requires SSO.** |
| Dataverse | SSO, OAuth2 | Can't use a gateway (MFA unsupported). |
| Enter data | N/A | Data embedded in the report (good for demos/POCs). |

**Via Enterprise gateway (and VNet gateway where noted):** SQL Server (SSO), SQL Server Analysis Services, Oracle (SSO), Teradata (SSO), ODBC. Snowflake/Databricks are reachable via the **Get Data (Power Query Online)** experience in Report Builder.

> RLS in paginated reports only works when the data source uses **SSO** authentication.
