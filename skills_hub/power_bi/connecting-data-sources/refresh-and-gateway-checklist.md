# Refresh & Gateway Checklist

All behaviors, limits, and steps below are confirmed in `connect-data/refresh-data.md`, `refresh-scheduled-refresh.md`, and `incremental-refresh-overview.md`. Do not assert refresh/gateway behavior beyond these docs.

## Contents
- Does this model even need a data refresh?
- Refresh frequency limits and durations
- Gateway decision and role
- Scheduled refresh checklist
- Incremental refresh recipe (large fact tables)
- Verification and monitoring
- Refresh pitfalls

---

## Does this model even need a data refresh?

Only **Import** mode semantic models import data and therefore require a **data refresh**. Others query the source at report time:

| Storage mode | Data refresh | Tile refresh | Notes |
|--------------|--------------|--------------|-------|
| Import | **Yes** (scheduled + on demand) | Automatic + on demand | A point-in-time copy; refresh to pick up source changes |
| DirectQuery | Not applicable | Automatic + on demand | Tiles refresh ~hourly (15 min to weekly); no data refresh |
| Live connection | Not applicable | Automatic + on demand | Report visuals cached; Refresh button requeries the model |
| Push | Not applicable | Automatic + on demand | Data pushed in by an external process |
| Direct Lake | Reframing (auto by default) | — | Disable auto-sync in scheduled refresh settings to control |

Composite models combining Import + DirectQuery refresh the Import/Dual tables; their "Scheduled refresh" lives under **Optimize Performance**.

## Refresh frequency limits and durations

| Capacity / license | Scheduled refreshes per day | Max refresh duration |
|--------------------|------------------------------|----------------------|
| Power BI Pro (shared capacity) | Up to **8** | **2 hours** |
| PPU and Premium / Fabric capacity (F / EM / P) | Up to **48** | **5 hours** (XMLA refresh can bypass) |

- The 8/day shared-capacity limit applies to scheduled **and** REST API refreshes; **manual UI on-demand** refreshes don't count toward the 8.
- The scheduler targets starting within **15 minutes** of the slot (can begin ~5 min early; up to 1 hour late under resource pressure).
- **No native monthly interval** — use Power Automate (or Fabric Data Factory pipelines) for monthly/cascading cadence.
- Power BI **deactivates the schedule after 4 consecutive failures** or an unrecoverable error (e.g., expired credentials) — threshold is not configurable.
- **Inactivity pause:** after **2 months** with no user visiting any report/dashboard on the model, scheduled refresh is paused; the owner is emailed. Resume by opening a report or running **Refresh now**.

## Gateway decision and role

No refresh succeeds unless the source is reachable. A gateway provides Power BI the connection info (endpoints, auth mode, credentials) for sources it can't reach directly.

```
Gateway decision:
- [ ] Source is a cloud source Power BI can reach directly (e.g., Azure SQL DB)?  -> No gateway. Manage creds in Data source credentials.
- [ ] Source is on-premises or in a virtual network?  -> Gateway required.
- [ ] You need to share data sources / multiple model owners?  -> Enterprise on-premises data gateway.
- [ ] You're the sole owner, no sharing, no enterprise gateway?  -> Personal mode gateway (limited).
- [ ] One mashup query merges on-prem + cloud sources?  -> Gateway used for the cloud source too.
```

Gateway roles and rules:
- **Enterprise on-premises data gateway (recommended):** gateway admin defines data source definitions (with credentials). The model owner doesn't supply credentials; only an admin can add data sources/users. You can request **Add to gateway** from the semantic model settings.
- **Personal mode gateway:** single-user, has documented limitations; you manage credentials yourself under **Data source credentials**. Exchange Online and HDFS aren't supported in personal mode.
- **One gateway per model:** a semantic model can use only **one** gateway connection — put **all** required data source definitions on the same gateway.
- **Cloud sources via gateway:** to refresh a mixed (gateway + cloud) mashup, either add a data source definition for the cloud source on the gateway, or enable **Allow user's cloud data sources to refresh through this gateway cluster**.
- **Sizing/reliability:** use a reliable enterprise gateway deployment; if you see gateway-unavailable/overloaded failures, scale up or scale out (add gateways to a cluster, or a new cluster). Use **separate gateways for Import vs DirectQuery/live** models so import refreshes don't degrade live query performance.

## Scheduled refresh checklist

To reach the screen: workspace > select the semantic model > **Refresh** > **Schedule refresh**.

```
Scheduled refresh setup:
- [ ] Gateway connection selected (if on-prem/VNet source)
- [ ] Data source credentials set (first refresh only; re-enter if password changes)
- [ ] "Configure a refresh schedule" toggled On
- [ ] Frequency + time slots set (<= 8/day Pro, <= 48/day PPU/Premium)
- [ ] Time zone set (affects slot times and incremental "current date")
- [ ] Failure notifications: keep owner email on; add backup recipients (must be in your Entra tenant)
- [ ] Schedule placed in an off-peak window
- [ ] Ran Refresh now to test; checked Refresh history
```

Notes:
- For on-premises SharePoint refresh, only **Anonymous, Basic, Windows (NTLM/Kerberos)** auth is supported (no ADFS / forms-based).
- **Query parameters** are supported for Import only (not DirectQuery/live). Configure them in semantic model settings.
- **Dynamic data sources** generally can't be refreshed in the service (exceptions: `Web.Contents` with `RelativePath`/`Query`, and queries referencing Power Query parameters). Check **Data source settings > Data sources in current file** for the warning.
- **Privacy settings** aren't published from Desktop — reapply them on the data source definitions in the service.

## Incremental refresh recipe (large fact tables)

Use for large/growing fact tables to refresh only recent partitions (rolling-window pattern). Supported on Pro, PPU, Premium, Embedded; **real-time DirectQuery partition (Hybrid) is Premium/PPU/Embedded only**.

Steps (all in Power BI Desktop, then publish):

1. **Create parameters** — in Power Query **Manage Parameters**, add two date/time parameters with the reserved, case-sensitive names `RangeStart` and `RangeEnd`.
2. **Filter the date column** by referencing the parameters (you must add the filter steps manually — the Custom Filter UI won't reference them):

   ```powerquery-m
   #"Filtered Rows" = Table.SelectRows(Source, each [OrderDate] >= RangeStart and [OrderDate] < RangeEnd)
   ```

   For an integer `yyyymmdd` surrogate date key:

   ```powerquery-m
   #"Filtered Rows" = Table.SelectRows(Source, each [OrderDateKey] >= Int32.From(DateTime.ToText(RangeStart,[Format="yyyyMMdd"]))),
   #"Filtered Rows1" = Table.SelectRows(#"Filtered Rows", each [OrderDateKey] < Int32.From(DateTime.ToText(RangeEnd,[Format="yyyyMMdd"])))
   ```

3. **Verify folding** — the filter must fold into the native source query. Reliable for SQL-based sources (SQL Database, Synapse, Oracle, Teradata); for others, confirm with Power Query Diagnostics or a source trace. A non-folding pure-Import policy makes the engine pull all rows locally, defeating the purpose. If the policy includes real-time DirectQuery, **non-folding transforms aren't allowed**.
4. **Define the policy** — in **Incremental refresh and real-time data**, set:
   - **Archive data starting before refresh date** (historical store period).
   - **Incrementally refresh data starting before refresh date** (refresh period — keep it the minimum needed).
   - Optional: **Get the latest data in real time with DirectQuery (Premium only)**, **Only refresh complete days** (required if real-time is on), **Detect data changes** (a different audit date/time column).
5. **Publish, then run the initial refresh** (manual, so you can watch it — it's slow; partitions are created and history loaded). Subsequent refreshes are fast (only recent partitions).
6. **Set a CommandTimeout** in the source step if needed:

   ```powerquery-m
   Source = Sql.Database("myserver.database.windows.net", "AdventureWorks", [CommandTimeout=#duration(0, 2, 0, 0)])
   ```

Caveats:
- After publishing an incremental-refresh model, you **can't republish from Desktop** (it wipes partitions/data) and **can't download the .pbix back**. Use ALM Toolkit / TMSL for later schema changes (Premium + XMLA endpoint).
- Real-time-DirectQuery models can only publish to a **Premium** workspace. Enable **Large semantic model storage format** before the first refresh if the model may exceed 1 GB.
- For reports on a real-time policy, enable **automatic page refresh** so visuals show the latest data.

## Verification and monitoring

- **Refresh now** on the semantic model settings page for an on-demand test (doesn't affect the next scheduled time).
- **Refresh history** shows success/failure, retry attempts, and separates **Data** vs **Query Cache** operations (data runs first; query cache only if data succeeds).
- A small warning icon on the workspace semantic-model list flags errors.
- **Fabric Monitoring Hub** centrally tracks refresh activities; **Refresh cancellation** is available for Import/Composite models on Premium/PPU/Embedded.
- Retrieve history programmatically via the Power BI REST API (`getrefreshhistoryingroup`).

## Refresh pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| On-prem source, no gateway | Can't enable schedule / refresh fails | Configure enterprise gateway + data source definition |
| Two gateways for one model | "single gateway connection" error | Consolidate all definitions on one gateway |
| Schema changed at source (Import) | Service data refresh fails | Schema refresh in Desktop, then republish |
| Dynamic data source | Refresh fails in service | Use RelativePath/Query or Power Query parameters |
| Non-folding incremental filter | Slow refresh / resource exhaustion | Make the date filter fold; verify with diagnostics |
| Republished incremental model from Desktop | Partitions and data wiped | Use ALM Toolkit / TMSL after first publish |
| Refreshes clustered at peak | Delays / skipped cycles | Spread refreshes across an off-peak window |
| 4 consecutive failures | Schedule deactivated | Fix root cause (often credentials), re-enable |
| Model inactive 2 months | Schedule paused, owner emailed | Open a report or Refresh now, re-enable schedule |
