---
name: connecting-data-sources
description: Brings data into Power BI by choosing a connector, deciding a storage mode (Import, DirectQuery, Dual, Composite, Direct Lake, Live connection), and configuring refresh and gateways. Use when connecting Power BI Desktop to a data source via Get data, picking or changing a table's storage mode, troubleshooting "should this be Import or DirectQuery", setting up scheduled or incremental refresh, deciding whether an on-premises data gateway is required, or designing a composite model that blends multiple sources. Covers connector categories, storage-mode tradeoffs (freshness, model size, performance, limitations), DirectQuery folding/limits, refresh frequency limits, and gateway roles.
---

# Connecting Data Sources in Power BI

## Overview

Getting data is the entry point of the Power BI authoring chain. Three decisions made here shape everything downstream: **which connector**, **which storage mode**, and **how the model stays current (refresh + gateway)**. A wrong storage-mode choice is expensive to undo — you cannot globally flip an Import model to DirectQuery — so decide deliberately before loading large tables.

**Default rule:** Use **Import** unless a specific constraint (data freshness, data volume, source-enforced security, or data sovereignty) forces DirectQuery, Direct Lake, or a Composite model. Import uses Power BI's in-memory VertiPaq engine and provides the richest feature set and best interactive performance.

This is a Phase 4 (Data Pipeline) skill with no dependencies. It feeds `transforming-data-power-query` (shaping the queries you create here) and `modeling-semantic-data` (relationships across the tables you load).

## When to Use

Trigger this skill when the user is:
- Connecting Power BI Desktop to any source via **Get data** (File, Database, Azure, Online Services, Microsoft Fabric, Other).
- Choosing or changing a table's **storage mode**, or asking "Import vs DirectQuery vs Composite".
- Setting up **scheduled refresh**, **incremental refresh**, or asking how often a model can refresh.
- Deciding whether an **on-premises data gateway** is needed and which kind.
- Building a **composite model** that blends multiple sources or extends a published semantic model.

## Workflow

Copy this checklist and work top to bottom:

```
Connect-data progress:
- [ ] Step 1 — Identify the source(s) and pick the connector
- [ ] Step 2 — Choose the storage mode (Import default; escalate only on a constraint)
- [ ] Step 3 — Connect: Get data > select > Load or Transform Data; set data types
- [ ] Step 4 — If multiple sources / mixed modes: design the composite model
- [ ] Step 5 — Decide gateway need (any on-prem or VNet source -> gateway)
- [ ] Step 6 — Configure refresh (scheduled; add incremental for large fact tables)
- [ ] Step 7 — Verify: run an on-demand refresh, check refresh history
```

### Step 1 — Pick the connector

In Power BI Desktop, **Home** ribbon > **Get data** opens the **Common data sources** list; choose **More** for the full **Get Data** dialog. Sources are grouped into categories: **File, Database, Microsoft Fabric, Power Platform, Azure, Online Services, Other** (plus **All**). For a source-to-connector quick reference and category contents, see [connector-quick-table.md](connector-quick-table.md).

Notes that matter at connect time:
- Sources marked **Beta** or **Preview** have limited support — don't use in production.
- To streamline connecting for other authors, export a **PBIDS** file (**File > Options and settings > Data source settings > Export PBIDS**). A PBIDS file holds one source and can pin `mode` to `Import` or `DirectQuery`.
- You can't currently connect to custom data sources secured with Microsoft Entra ID.

### Step 2 — Choose the storage mode

This is the highest-leverage decision. Use the full decision matrix in [storage-mode-decision-matrix.md](storage-mode-decision-matrix.md). Quick summary of the available modes:

| Mode | Data location | Use when |
|------|---------------|----------|
| **Import** | Cached in-memory (VertiPaq) | Default. Best performance + full feature set. |
| **DirectQuery** | Stays in source, queried per visual | Data too large/volatile to import, source-enforced security (SSO), or sovereignty rules. |
| **Dual** | Both (cache or query) | Dimension tables queried alongside DirectQuery facts from the **same source** — reduces limited relationships. |
| **Direct Lake** | Scanned from OneLake delta tables | Fabric lakehouse/warehouse scale with low-latency reads (Fabric only). |
| **Live connection** | Remote semantic model / Analysis Services | Reuse a central governed model; no local model (a "thin report"). |
| **Composite** | Mixed modes / multiple sources | Blend sources, or extend a published semantic model with extra data. |

Key constraints (confirmed in docs):
- You **cannot** globally switch an Import-only model to DirectQuery. You can convert a **DirectQuery** table to Import or Dual, but **not back** to DirectQuery (except in Power BI web modeling / live edit, which have version control).
- A table created via a DirectQuery-capable connection can switch between DirectQuery, Import, and Dual **only if all applied transformations still fold**.
- DirectQuery has a **1,000,000-row intermediate result limit** and a **4-minute per-query timeout** in the service. Aim for visual queries under 5 seconds.

### Step 3 — Connect and set data types

1. Select the source, **Connect**, supply credentials/connection info.
2. In **Navigator**, select tables/views. Choose **Load** (load now) or **Transform Data** (open Power Query Editor first — preferred, hand off to `transforming-data-power-query`).
3. Set correct **data types** in Power Query Editor. Critical cases (see [storage-mode-decision-matrix.md](storage-mode-decision-matrix.md) for the DirectQuery-specific ones):
   - Set Boolean columns to **True/False** (otherwise the service shows `-1`/`0` after refresh).
   - Use `Text.Trim` (or **Transform > Format > Trim**) to remove leading/trailing spaces — leading spaces aren't auto-trimmed and break relationships.
   - Remove **Binary** columns before load; binary isn't supported in the model and breaks gateway refresh.
   - For money where rounding matters, prefer **Fixed decimal number** over **Decimal number** (approximate floating point).

### Step 4 — Composite model design (if applicable)

A **composite model** has tables in more than one storage mode, or DirectQuery tables from different sources. A model of only Import (or only Direct Lake) tables is **not** composite.

- Cross-source relationships default to **many-to-many** cardinality and are always **limited relationships** (INNER JOIN semantics — unmatched rows drop out). Keep cross-source-group join columns **low cardinality (≈ < 50,000 unique values)**; avoid large string keys.
- Set **dimension** tables to **Dual** when queried with DirectQuery facts from the same source; set large/volatile **fact** tables to **DirectQuery**.
- **Security warning:** queries can push data values from one source into a query sent to another source. Composite-on-published-semantic-models needs **Build** permission on each model in the chain (max chain length 3).
- SAP HANA and SAP Business Warehouse (as multidimensional) **cannot** be used in composite models.

See [storage-mode-decision-matrix.md](storage-mode-decision-matrix.md) for source-group and Dual-propagation details.

### Step 5 — Decide the gateway

No data refresh succeeds unless the source is reachable. Decide by source location:

- **Cloud source** Power BI can reach directly (e.g., Azure SQL DB): **no gateway**; manage credentials under **Data source credentials**.
- **On-premises or virtual-network source:** **gateway required**. Prefer the **enterprise on-premises data gateway** (shared, admin-managed data source definitions) over **personal mode** (single user, limited).
- **Mixed mashup** that merges/appends an on-prem source with a cloud source in one query: the gateway is used for the cloud source too. A semantic model can use **only one** gateway connection — put all required data source definitions on the same gateway.

Full gateway role and refresh setup: [refresh-and-gateway-checklist.md](refresh-and-gateway-checklist.md).

### Step 6 — Configure refresh

Only **Import** mode requires a data refresh. DirectQuery / Direct Lake / Live connection query the source at report time (Direct Lake "reframes"); they still do tile/visual refreshes.

- **Scheduled refresh:** Power BI Pro = up to **8/day**; PPU and Premium/Fabric capacity = up to **48/day**. Set the time zone; enable failure notifications.
- **Incremental refresh:** for large/growing fact tables. Define `RangeStart` / `RangeEnd` (reserved, case-sensitive date/time parameters) in Power Query, filter the date column with them, then set the policy in **Incremental refresh and real-time data**. Requires **query folding** of the filter. Real-time DirectQuery partition (Hybrid) is **Premium/PPU/Embedded only**.
- Refresh duration limit: **2 hours** shared capacity, **5 hours** Premium.

See [refresh-and-gateway-checklist.md](refresh-and-gateway-checklist.md) for the full checklist and the incremental-refresh recipe.

### Step 7 — Verify

Run **Refresh now** on the semantic model settings page, then check **Refresh history** for success/failure and timing. Schedule refreshes for off-peak windows.

## Key Concepts

- **Storage mode is per table.** A "model's mode" is just the mix of its tables' modes. Many DirectQuery limitations apply per table by storage mode; some apply model-wide if any table is DirectQuery.
- **Query folding** is the make-or-break for DirectQuery and incremental refresh: transformations must collapse into one native source query. Confirm with **View Native Query** in Power Query Editor.
- **Regular vs limited relationships:** intra-source-group 1:1/1:* relationships are regular; cross-source and *:* are limited (INNER JOIN, rows without a match on both sides disappear).
- **Refresh ≠ schema refresh.** In the service, a renamed/removed source column fails data refresh because no schema refresh runs there; fix the schema in Desktop and republish.

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Choosing DirectQuery by default | Slow visuals, 1M-row / 4-min timeouts | Use Import unless a real constraint forces otherwise |
| Expecting to flip Import -> DirectQuery later | No global toggle exists | Decide before loading; or add DirectQuery tables / go composite |
| Non-folding Power Query step on a DirectQuery table | Error or no incremental benefit | Keep steps foldable; check **View Native Query** |
| Relative-date filter in Power Query (DirectQuery) | Filter becomes a fixed literal date | Use relative-date columns in the date table / model layer |
| On-prem source with no gateway | Refresh fails / can't enable schedule | Configure an enterprise gateway with the data source definition |
| Multiple gateways for one model | "single gateway connection" error | Put all data source definitions on one gateway |
| Boolean column left as Any/Text | Shows `-1`/`0` after service refresh | Set column type to **True/False** in Desktop, republish |
| Leading spaces / binary columns | Relationship cardinality errors, gateway refresh errors | `Text.Trim`; remove binary columns as last query step |
| High-cardinality cross-source join | Huge native queries, failures | Low-cardinality keys; raise relationship granularity |
| Republishing an incremental-refresh model from Desktop | Existing partitions/data wiped | After first publish, use ALM Toolkit / TMSL for schema changes |

## Success Criteria

- Connector chosen from a confirmed category; Beta/Preview avoided in production.
- Storage mode justified by the decision matrix (Import unless a stated constraint).
- Data types correct (True/False, trimmed text, no binary, fixed-decimal money where needed).
- Gateway provisioned for any on-prem/VNet source; one gateway per model.
- Refresh configured within capacity limits; incremental refresh for large fact tables with folding verified.
- A successful on-demand refresh confirmed in Refresh history.

## Related Skills

- `transforming-data-power-query` — shapes the queries created here (folding, star-schema shaping).
- `modeling-semantic-data` — relationships, cardinality, and Dual/composite relationship behavior.
- `optimizing-model-performance` — data reduction, aggregations over DirectQuery, refresh tuning.
- `securing-row-level-access` — RLS interacts with DirectQuery SSO and composite/source-model security.

## Reference Files

- [storage-mode-decision-matrix.md](storage-mode-decision-matrix.md) — Import/DirectQuery/Dual/Direct Lake/Live/Composite tradeoffs, when-to-use, limitations, conversion rules.
- [connector-quick-table.md](connector-quick-table.md) — connector-by-source quick table and category contents.
- [refresh-and-gateway-checklist.md](refresh-and-gateway-checklist.md) — scheduled/incremental refresh, gateway role and sizing, verification.

---
*Provenance: built per SKILL 11 "Connecting Data Sources" in `power_bi_user/planning/01-POWERBI-FULL-LIFECYCLE-SKILLS-PLAN.md`; grounded solely in the Power BI docs `connect-data/desktop-connect-to-data.md`, `desktop-data-sources.md`, `desktop-directquery-about.md`, `desktop-data-types.md`, `refresh-data.md`, `refresh-scheduled-refresh.md`, `incremental-refresh-overview.md`, `transform-model/desktop-storage-mode.md`, `desktop-composite-models.md`, and `guidance/directquery-model-guidance.md`, `composite-model-guidance.md`.*
