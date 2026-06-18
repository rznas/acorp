# Connector Quick Table (by source)

All connectors and categories below are confirmed in `connect-data/desktop-data-sources.md`. The full live list is at `connect-data/power-bi-data-sources.md` (Microsoft maintains it). Connectors marked **(Beta)** / **(Preview)** have limited support — don't use in production. Some connectors must be enabled under **File > Options and settings > Options > Preview features**.

## Contents
- How to open Get data
- Get data categories
- Quick table: common source -> connector + category + typical mode
- File connectors
- Database connectors
- Azure connectors
- Microsoft Fabric / Power Platform connectors
- Online Services and Other connectors
- PBIDS files

---

## How to open Get data

**Home** ribbon > **Get data** label/arrow -> **Common data sources** list. Choose **More** for the full **Get Data** dialog. Or click the **Get data** icon directly. After selecting a source: **Connect** -> enter connection info/credentials -> in **Navigator** pick tables/views -> **Load** or **Transform Data**.

## Get data categories

The **Get Data** dialog groups sources into: **All, File, Database, Microsoft Fabric, Power Platform, Azure, Online Services, Other**. The **All** category lists everything.

## Quick table: common source -> connector + category + typical mode

"Typical mode" follows the default-Import rule; DirectQuery is available for many relational/analytic sources (see `storage-mode-decision-matrix.md`). Live connection applies to existing semantic models / Analysis Services.

| You want to connect to | Connector (as listed) | Category | Typical mode |
|------------------------|-----------------------|----------|--------------|
| Excel file | Excel Workbook | File | Import |
| CSV / text file | Text/CSV | File | Import |
| JSON / XML / Parquet file | JSON / XML / Parquet | File | Import |
| A folder of files | Folder | File | Import |
| SharePoint document folder | SharePoint folder | File | Import |
| SQL Server (on-prem) | SQL Server database | Database | Import or DirectQuery (gateway for refresh) |
| Oracle / Db2 / MySQL / PostgreSQL / Teradata | (each listed by name) | Database | Import or DirectQuery |
| Snowflake / Google BigQuery / Amazon Redshift / Vertica | (each listed by name) | Database | Import or DirectQuery |
| SAP HANA / SAP BW | SAP HANA database / SAP Business Warehouse (App or Message Server) | Database | Import or DirectQuery (not in composite as multidimensional) |
| SQL Server Analysis Services | SQL Server Analysis Services database | Database | Live connection / DirectQuery |
| Azure SQL Database | Azure SQL Database | Azure | Import or DirectQuery (no gateway if cloud-reachable) |
| Azure Synapse SQL | Azure Synapse Analytics SQL | Azure | Import or DirectQuery |
| Azure Databricks | Azure Databricks | Azure | Import or DirectQuery |
| Azure Data Lake Storage Gen2 / Blob | Azure Data Lake Storage Gen2 / Azure Blob Storage | Azure | Import |
| Azure Data Explorer (Kusto) | Azure Data Explorer (Kusto) | Azure | Import or DirectQuery |
| Existing Power BI semantic model | Power BI semantic models | Microsoft Fabric | Live connection (or DirectQuery via "Make changes to this model") |
| Fabric Warehouse / Lakehouse | Warehouses / Lakehouses | Microsoft Fabric | Direct Lake / Import / DirectQuery |
| Dataflows | Dataflows | Microsoft Fabric / Power Platform | Import |
| Dataverse | Dataverse | Power Platform | Import or DirectQuery |
| SharePoint Online list | SharePoint Online List | Online Services | Import |
| Salesforce | Salesforce Objects / Salesforce Reports | Online Services | Import |
| Dynamics 365 (Dataverse) | Dynamics 365 (Dataverse) | Online Services | Import |
| A web page / REST endpoint | Web | Other | Import |
| OData service | OData Feed | Other | Import |
| Generic ODBC / OLE DB | ODBC / OLE DB | Other | Import or DirectQuery (driver-dependent) |
| R / Python output | R script / Python script | Other | Import |
| Start from scratch | Blank Query | Other | Import |

## File connectors

Excel Workbook, Text/CSV, XML, JSON, Folder, PDF, Parquet, SharePoint folder.

## Database connectors

SQL Server, Access, SQL Server Analysis Services, Oracle, IBM Db2, IBM Informix (Beta), IBM Netezza, MySQL, PostgreSQL, Sybase, Teradata, SAP HANA, SAP Business Warehouse (Application Server / Message Server), Amazon Redshift, Impala, Google BigQuery (and Microsoft Entra ID variant), Vertica, Snowflake, Essbase, AtScale Models, Amazon Athena, MariaDB, MarkLogic, MongoDB Atlas SQL, Denodo, Dremio (Software / Cloud), Exasol, plus several Beta/Preview connectors (Actian, ClickHouse, KX kdb Insights, Kyvos ODBC, etc.).

> Some database connectors require enabling under **Preview features**.

## Azure connectors

Azure SQL Database, Azure Synapse Analytics SQL, Azure Analysis Services database, Azure Database for PostgreSQL, Azure Blob Storage, Azure Table Storage, Azure Cosmos DB (v1/v2), Azure Data Explorer (Kusto), Azure Data Lake Storage Gen2, Azure HDInsight (HDFS / Spark / Interactive Query), Azure Cost Management, Azure Resource Graph, Azure Databricks, Azure Synapse Analytics workspace (Beta).

## Microsoft Fabric / Power Platform connectors

- **Microsoft Fabric:** Power BI semantic models, Dataflows, Datamarts (preview), Warehouses, Lakehouses, KQL Databases.
- **Power Platform:** Power BI dataflows (Legacy), Common Data Service (Legacy), Dataverse, Dataflows.

## Online Services and Other connectors

- **Online Services** (selection): SharePoint Online List, Microsoft Exchange Online, Dynamics 365 (Dataverse), Dynamics 365 Business Central, Salesforce Objects/Reports, Google Analytics, Adobe Analytics, Databricks, plus many partner connectors (numerous Beta).
- **Other** (selection): Web, SharePoint list, OData Feed, Active Directory, Microsoft Exchange, Hadoop File (HDFS), Spark, Hive LLAP, R script, Python script, ODBC, OLE DB, Google Sheets, Delta Sharing, FHIR, Blank Query, plus many partner connectors.

> **Template Apps**: available via the **Template Apps** link near the bottom of the Get data window (varies by organization).
> You can't currently connect to custom data sources secured with Microsoft Entra ID.

## PBIDS files

A **PBIDS** (`.pbids`) file streamlines connecting for other authors — it holds **one** data source and can pin `mode` to `Import` or `DirectQuery` (if omitted/null, the author is prompted). Create it via **File > Options and settings > Data source settings > Export PBIDS** (recommended) or hand-edit JSON. PBIDS holds no authentication, table, or schema info; Live Connect, Wiki URLs, and Blank Query aren't supported in PBIDS.

Example (SQL Server, DirectQuery):

```json
{
  "version": "0.1",
  "connections": [
    {
      "details": {
        "protocol": "tds",
        "address": { "server": "server-name-here", "database": "db-name-here (optional)" }
      },
      "options": {},
      "mode": "DirectQuery"
    }
  ]
}
```
