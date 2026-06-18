# Usage metrics & saved views

Full configuration reference for usage metrics, MCP automation, and saved views. Source: PostHog Customer Analytics docs (usage-metrics, create-usage-metrics, saved-views).

## Table of contents

- [Usage metrics overview](#usage-metrics-overview)
- [Creating a usage metric](#creating-a-usage-metric)
- [Configuration fields](#configuration-fields)
- [Calculation, display, filters](#calculation-display-filters)
- [Data Warehouse metrics](#data-warehouse-metrics)
- [Managing usage metrics via MCP](#managing-usage-metrics-via-mcp)
- [Saved views](#saved-views)

## Usage metrics overview

Usage metrics define a global activity measurement and track it per customer. Any event your users perform — or rows from a [Data Warehouse](/docs/data-warehouse) table — can become a usage metric. They appear on customer profiles and in customer lists, surfacing **expansion opportunities** (rising usage) and **churn risk** (falling usage).

Example SaaS metrics:

| Metric | Events to match | Use case |
| --- | --- | --- |
| API calls | `api_request` | Identify heavy API users for expansion |
| Feature adoption | `feature_x_used` | Track adoption of new features |
| Error rate | `$exception` | Spot customers hitting issues |
| Export usage | `export_created` | Find power users |

## Creating a usage metric

Two entry points:

1. **Settings > Customer Analytics > Add usage metric**
2. Open any [customer profile](/docs/customer-analytics/customer-profiles) and click **Add usage metric**

## Configuration fields

| Field | Description |
| --- | --- |
| **Name** | Column title shown in customer lists |
| **Interval** | Time period: 7, 30, or 90 days |
| **Format** | Numeric or currency |
| **Display** | Number (default) or Sparkline |
| **Calculation** | Count of events/rows (default) or Sum of property/column |
| **Property to sum** | Numeric property/column to sum (only when Calculation = Sum) |
| **Match events** | Which events or Data Warehouse table is the source |
| **Filters** | Extra conditions to narrow matched events (events source only) |

## Calculation, display, filters

**Calculation**
- **Count of events** (or **Count of rows** for DW) — total matched events/rows. Default.
- **Sum of property** (or **Sum of column**) — sums a numeric event property or table column. Useful for API tokens consumed, storage used, revenue generated. Reveals a **Property/Column to sum** field.

**Display**
- **Number** — single value with a percentage change indicator. Default.
- **Sparkline** — bar chart of daily values over the interval, plus total and % change. Best for spotting trends / declining accounts at a glance.

**Match events**
- Events-based: add multiple event matchers; the metric counts events matching *any* of them (e.g. `api_request` OR `api_call`).
- Data Warehouse: select one table and set the timestamp column and group key column.
- A metric uses a single source — you can't mix events and a DW table in the same metric.

**Filters** — apply to all matched events; events-based only. Narrow what counts, e.g. only successful API calls via `status = 200`.

## Data Warehouse metrics

Use when usage/billing data lives outside PostHog (Stripe, your own DB). Specify:

- **Table** — the DW table to query
- **Timestamp column** — used to filter rows by the metric's date range
- **Group key column** — used to match rows to the correct customer

Each DW metric is limited to a single table.

> Data Warehouse usage metrics currently render only on **group** profiles, not person profiles.

## Managing usage metrics via MCP

The [PostHog MCP server](/docs/model-context-protocol) lets AI agents (Cursor, Claude Code, VS Code) manage usage metrics programmatically:

| Tool | Description |
| --- | --- |
| `usage-metrics-create` | Create a new usage metric |
| `usage-metrics-list` | List all usage metrics in the project |
| `usage-metrics-retrieve` | Get a single usage metric by ID |
| `usage-metrics-partial-update` | Update an existing usage metric |
| `usage-metrics-destroy` | Delete a usage metric |

Example prompts: "List all usage metrics in my project." / "Create a usage metric that counts `api_request` events over the last 30 days." / "Update my API calls metric to use a 90-day interval."

## Saved views

Saved views store a reusable configuration for the **Persons** and **Groups** lists, so you can switch between segmentation lenses without rebuilding the table.

Each view stores:
- **Column configuration** — which columns are visible and their order
- **Filters** — active filters on the list
- **Sort order** — current sort

Views are saved per table and can be **private or shared** with project members.

Create:
1. **People & groups > Persons**
2. **Configure columns** in the toolbar above the table
3. Add filters via **Filter**
4. **Save current view** and name it

Once you have one view, the button becomes a dropdown (switch views, or **Create new view...**). Switching a view applies its columns, filters, and sort. Changing an active view shows an **Update** button to save edits. The gear icon next to a view name lets you **Rename** or **Delete**.

Example: a "High value customers" view showing lifetime value, last active date, and subscription status columns, filtered for revenue above a threshold — combine usage-metric columns with filters to surface at-risk or expansion-ready accounts in one click.
