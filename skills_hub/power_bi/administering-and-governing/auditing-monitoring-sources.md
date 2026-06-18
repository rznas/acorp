# Auditing and Monitoring Sources

Catalog of Power BI / Fabric auditing and monitoring data sources, what each answers, the level it serves (**tenant / report / data**), and how to access it. Grounded in the implementation-planning auditing-monitoring and admin-activity-log docs.

**Auditing** = understanding the system and user activity (manual + automated). **Monitoring** = ongoing alerting/automation about what is happening now.

## Contents
- Why audit (use cases)
- Sources table (by level)
- Activity log: access methods, limits, ELT pattern
- Tenant inventory
- Monitoring and alerting
- Report-level and data-level auditing
- Worked PowerShell extraction example

---

## Why audit

Auditing data supports: **adoption analysis** (organizational/user/solution), **usage patterns** (content usage, active users/creators), **published-items inventory** (lineage, model-to-report ratio, unused content, oversharing), **educate/support users**, **mitigate risk** (governance/security, tenant-setting changes, unknown data sources), **compliance** (regulatory/organizational/contractual/internal/external audit requests), **license & cost management** (license mix, capacity sizing, Autoscale, chargebacks, trials), and **performance** input.

> Priority: if you are not already extracting and storing **user activity data**, make it urgent — the activity log retains only **28 days**.

## Sources table (by level)

| Source | Level | What it answers | How to access |
|--------|-------|------------------|---------------|
| **Power BI activity log** | Tenant | Who did what, when (ViewReport, ShareReport, CreateApp, UpdatedAdminFeatureSwitch, etc.); ~30+ days of behavior when stored | `Get-PowerBIActivityEvent` (PowerShell, 1 day/call) or **Get Activity Events** admin REST API (1 hour/call). Fabric admin or service principal. 28-day window. |
| **Unified audit log (Microsoft Purview)** | Tenant | Power BI events alongside other M365 services; supports alert policies | Microsoft Purview / Microsoft 365 audit log; alert policies for events such as `UpdatedAdminFeatureSwitch` |
| **Tenant inventory (metadata snapshot)** | Tenant | What exists: workspaces, reports, semantic models, lineage, data sources, gateways, capacities; permissions | **Metadata scanning (scanner) APIs** (incremental — large tenants); **Get Groups As Admin** / `Get-PowerBIWorkspace` (full pull — small tenants) |
| **Get Tenant Settings REST API** | Tenant | Current value of every tenant setting; snapshot for diffing | `/rest/api/fabric/admin/tenants/list-tenant-settings`; snapshot weekly and diff |
| **Admin monitoring workspace** | Tenant | Pre-built reports on usage and activity | Fabric admin monitoring workspace |
| **Admin APIs (various)** | Tenant | Compliance checks: RLS on models (Get Datasets As Admin), report sharing (Get User Artifact Access As Admin), workspace roles (Get Group Users As Admin), app permissions (Get App Users As Admin), capacity region (Get Capacities) | Power BI / Fabric admin REST APIs |
| **Usage metrics report** | Report | Which users use a report, how often, on which devices (report creators' self-service view) | Per-report usage metrics in the service (report-level auditing) |
| **Semantic model event logs (Log Analytics)** | Data | Analysis Services engine activity, performance, query trends for a semantic model | **Azure Log Analytics** integration (enabled via Azure connections / workspace setting) |
| **Refresh & query telemetry** | Data | Refresh operations, live-connection queries, composite-model dependencies, query caching, where query folding fails, active DirectQuery connections, storage modes used | Data-level auditing (activity log + Log Analytics + model metadata) |
| **Gateway logs & monitoring** | Data | Gateway uptime/health, query failures, slow refreshes, gateway prevalence/oversharing | Gateway machine logs, Power Platform admin center, gateway PowerShell cmdlets, activity log (see `connecting-data-sources`) |
| **Microsoft Defender for Cloud Apps** | Tenant | Real-time control events and policy matches (for example, blocked downloads of labeled content) | Defender for Cloud Apps policies/alerts |

## Activity log: access methods, limits, ELT pattern

- **Two access methods**: `Get-PowerBIActivityEvent` cmdlet (recommended) or the **Get Activity Events** REST API directly. The cmdlet returns **one day per call**, handles continuation tokens, and refreshes the access token automatically; the raw API returns **one hour per call** and you manage tokens/pagination yourself.
- **Permissions**: the authenticating user must be a **Fabric administrator** (it is an admin API). For scheduled/unattended runs, use a **service principal**.
- **Window**: only the last **28 days** are available — export and retain.
- **Throttling**: **200 requests/hour** on Get Activity Events. Do not loop/filter during extraction.
- **ELT, not ETL**: keep extraction simple — pull the **raw JSON** once per day, store it, then transform/filter/format separately. Raw JSON returns all available fields (which differ per event type), is resilient to schema changes, and is easy to hand off to data engineers. Extracting the log is itself a logged operation (`ExportActivityEvents`); you may want to separate or omit admin activities when analyzing.
- **UTC**: audited timestamps are always UTC — build all auditing on UTC.

## Tenant inventory

A point-in-time metadata snapshot. Combine with activity-log data to:
- Find **unused content** (in the inventory but absent from recent activity) → candidates for retirement.
- Detect **oversharing** (correlate `Highly Restricted`-type labels with high permission counts).
- Measure **data reuse** (semantic-model-to-report ratio; many duplicate models → train on shared semantic models or consolidate).
- Identify **most-used data sources and gateways** via lineage.

Snapshot **weekly or monthly**; for large tenants use the incremental scanner APIs.

## Monitoring and alerting

- Alert on high-risk events using **Microsoft 365 / Purview alert policies** or **Defender for Cloud Apps** (for example, notify when `UpdatedAdminFeatureSwitch` fires or when an org-wide sharing link is created unexpectedly).
- Monitor **related Azure services** and **gateway machine health** with enterprise monitoring tools.
- If you have Fabric capacity, you can orchestrate a gateway/governance monitoring solution: Data Factory to land logs in OneLake, notebooks to call REST APIs, Power BI to report on health, and Fabric Activator (Data Activator) to alert owners on anomalies.

## Report-level and data-level auditing

- **Report-level**: report creators use **usage metrics** to see which users use their reports and how often — self-service auditing without admin rights.
- **Data-level**: data creators track refresh, query, and performance telemetry via the activity log and Log Analytics semantic-model event logs; use this as input to `optimizing-model-performance` (slow queries, failed query folding, DirectQuery connection counts, storage modes).

## Worked PowerShell extraction example

Export the previous full UTC day (avoids partial-day data); adapt the loop to back-fill up to 28 days. Use a service principal for scheduled runs.

```powershell
#Input values before running the script:
$NbrDaysToExtract  = 7
$ExportFileLocation = 'C:/Power-BI-Raw-Data/Activity-Log'   # folder must exist; no trailing slash
$ExportFileName     = 'PBIActivityEvents'
#--------------------------------------------
Connect-PowerBIServiceAccount   # interactive; swap for service-principal auth in production

#Start with yesterday so each export covers a full 24 hours:
[datetime]$DayUTC = ([datetime]::Today.ToUniversalTime().Date).AddDays(-1)
[string]$WrittenLabel = ([datetime]::Now.ToUniversalTime()).ToString('yyyyMMddHHmm')

For ($LoopNbr = 0; $LoopNbr -lt $NbrDaysToExtract; $LoopNbr++) {
    [string]$DateLabel = $DayUTC.AddDays(-$LoopNbr).ToString('yyyy-MM-dd')
    [string]$FullName  = "$ExportFileName-$($DateLabel -replace '-','')-$WrittenLabel.json"

    $Events = Get-PowerBIActivityEvent `
        -StartDateTime ($DateLabel + 'T00:00:00.000') `
        -EndDateTime   ($DateLabel + 'T23:59:59.999')

    $Events | Out-File "$ExportFileLocation/$FullName"
    Write-Verbose "File written: $FullName" -Verbose
}
```

Filter *after* extraction (for example, to one workspace) by converting the stored JSON and selecting fields — never re-query the API just to filter, because of the 200 req/hour limit.
