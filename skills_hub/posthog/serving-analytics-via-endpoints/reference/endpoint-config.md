# Endpoint configuration reference

Deep configuration detail for PostHog Endpoints. The SKILL.md overview links here. Use this when specifying or debugging variable rules, materialization compatibility, execution behavior, or integrations.

## Table of contents

- [Creating an endpoint (UI paths)](#creating-an-endpoint-ui-paths)
- [SQL variables in depth](#sql-variables-in-depth)
- [Insight magic variables in depth](#insight-magic-variables-in-depth)
- [Data freshness values](#data-freshness-values)
- [Materialization compatibility](#materialization-compatibility)
- [Timestamp bucketing](#timestamp-bucketing)
- [Execution: refresh and debug params](#execution-refresh-and-debug-params)
- [Response format](#response-format)
- [Running a specific version](#running-a-specific-version)
- [OpenAPI / SDK generation](#openapi--sdk-generation)
- [Per-customer analytics pattern](#per-customer-analytics-pattern)
- [Retool / internal tools integration](#retool--internal-tools-integration)
- [Performance & cost tips](#performance--cost-tips)
- [Troubleshooting](#troubleshooting)

## Creating an endpoint (UI paths)

**From an insight:** open the insight → info & actions (three dots, top right) → **Create endpoint**. Or Endpoints page → **New endpoint** → **Insight-based endpoint** → pick/create insight.

**From SQL:** SQL editor → **Save as endpoint** (top right).

Name rules: descriptive and stable (`daily_active_users`, `account_users_activity`, `signup_funnel`); avoid `endpoint1`, `test`, `query`. The name is part of the run URL.

## SQL variables in depth

Define with `{variables.name}` syntax in the query; create/manage in [data management variables](https://app.posthog.com/data-management/variables) (Variables tab in the SQL editor's output pane: **Name**, **Type** String/Number/etc., optional **Default value**).

- Each `{variables.X}` placeholder must have a matching variable whose `code_name` is `X`.
- **Variables are required at execution** unless they have a default. Missing required variable → `"Required variable 'X' not provided"`.

Example query and call:

```sql
SELECT toDate(timestamp) AS date, count() AS event_count
FROM events
WHERE properties.customer_id = {variables.customer_id}
  AND timestamp >= now() - INTERVAL 30 DAY
GROUP BY date ORDER BY date
```

```bash
curl -X POST \
  -H "Authorization: Bearer $POSTHOG_PERSONAL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"variables": {"customer_id": "acme-corp"}}' \
  "<ph_app_host>/api/projects/{project_id}/endpoints/customer_events_by_day/run"
```

## Insight magic variables in depth

No query editing needed — variables come from the insight config:

- **Breakdown variables:** each breakdown property becomes a variable. Insight broken down by `$browser` → pass `{"$browser": "Chrome"}`. Multiple breakdowns → pass each (`{"$browser":"Chrome","$os":"Mac OS X"}`); results match ALL values.
- **Date variables:** `date_from` / `date_to` filter the range — **non-materialized endpoints only**. Combine with breakdowns freely.
- Supported insight query types for breakdown variables: **TrendsQuery**, **RetentionQuery**. (Materialization also supports Lifecycle for result-format transformation.)

## Data freshness values

Single setting (Configuration tab → **Data freshness**) drives both cache TTL (non-materialized) and materialization schedule (materialized):

| Seconds | Duration |
| --- | --- |
| 900 | 15 minutes |
| 1,800 | 30 minutes |
| 3,600 | 1 hour |
| 21,600 | 6 hours |
| 43,200 | 12 hours |
| 86,400 | 1 day (default) |
| 604,800 | 7 days |

For materialized endpoints, cache TTL is computed dynamically as the time remaining until the next scheduled materialization run.

## Materialization compatibility

Enable in Configuration tab → **Materialization** toggle. First run happens immediately. Statuses: **Completed**, **Running**, **Failed** (falls back to direct execution until next success).

Cannot materialize when:

- Variables use unsupported operators or sit outside a `WHERE` clause.
- Endpoint is inactive (activate first).
- Query uses **cohort breakdowns** (complex union queries).
- Query uses **compare mode** (alters series count).

Variable compatibility:

| Variable usage | Materializable | Notes |
| --- | --- | --- |
| Equality `=` on columns | Yes | Standard column filters |
| Range `>=`,`>`,`<`,`<=` on timestamp columns | Yes | Uses timestamp bucketing |
| Variables in `HAVING` clauses | No | |
| Variables in `OR` conditions | No | |
| `LIKE` operator | No | |

Materialized endpoints **do not accept** `date_from`/`date_to` (data is pre-computed) — pass those would fail; only breakdown variables are allowed, and they are **required** for materialized insight endpoints.

## Timestamp bucketing

When a query uses timestamp range variables, PostHog pre-aggregates into time buckets and re-aggregates only the buckets within the requested range at read time.

- Granularities: `hour`, `day` (default, `toStartOfDay`), `week`, `month`. Configurable per timestamp column in the Configuration tab (dropdown appears when materialization is on and the query has timestamp range variables).
- Only re-aggregatable functions work: `count`, `sum`, `min`, `max`, and `*If` combinators (`countIf`, `sumIf`). NOT supported: `avg`, `uniq`, `countDistinct`.

## Execution: refresh and debug params

`POST .../endpoints/{endpoint_name}/run` body options:

| `refresh` | Behavior |
| --- | --- |
| `cache` (default) | Return cached results if fresh; else materialized results or direct query |
| `force` | Bypass cache; return materialized results or run against raw data |
| `direct` | Materialized endpoints only — bypass materialized results, run against raw data |

`"debug": true` adds the executed `hogql` query to the response — useful to confirm a request actually hit materialized storage.

## Response format

HTTP 200:

```json
{
  "results": [["value1", "value2"], ["value3", "value4"]],
  "columns": ["column_a", "column_b"]
}
```

`results` = array of rows (each an array of values); `columns` = column names in order; plus metadata fields (materialization status, version, etc.).

## Running a specific version

Latest version runs by default. Pin via `?version=2` (query string or body). Deactivate a single buggy version in the Versions tab (three dots → **Deactivate version**) without disabling the whole endpoint. Each version can be materialized independently.

```bash
curl -H "Authorization: Bearer $POSTHOG_PERSONAL_API_KEY" \
  "<ph_app_host>/api/projects/{project_id}/endpoints/{endpoint_name}/run?version=2"
```

## OpenAPI / SDK generation

```bash
curl -H "Authorization: Bearer $POSTHOG_PERSONAL_API_KEY" \
  "<ph_app_host>/api/projects/{project_id}/endpoints/{endpoint_name}/openapi.json" > openapi.json

# TypeScript
npx @hey-api/openapi-ts -i openapi.json -o ./src/posthog-client
# Python
openapi-generator generate -i openapi.json -g python -o ./posthog_client
```

Spec is OpenAPI 3.0; works with any compatible generator. Includes request variables, response schema, and auth requirements.

## Per-customer analytics pattern

1. SQL endpoint filtering on a per-customer property, e.g. `WHERE properties.user_id = {variables.user_id}`; define `user_id` (String).
2. Track events with that context: `posthog.capture("hedgehog_clicked", { user_id: currentUser.id })`.
3. Backend API route forwards the authenticated user's id as the variable (key stays server-side):

```ts
const response = await fetch(
  `https://us.posthog.com/api/projects/${PROJECT_ID}/endpoints/hedgehog_clicks_by_user/run`,
  { method: "POST",
    headers: { "Content-Type": "application/json",
               Authorization: `Bearer ${process.env.POSTHOG_API_KEY}` },
    body: JSON.stringify({ variables: { user_id } }) });
```

Security: never expose the key client-side; required variables prevent unfiltered leaks; validate the user is authorized for the requested id.

## Retool / internal tools integration

1. Retool → **Resources** → **Create new** → **REST API**. Base URL `https://<ph_app_host>/api/projects/{project_id}/endpoints`; header `Authorization: Bearer {personal_api_key}`.
2. Add a query on that resource: **POST**, URL path `/{endpoint_name}/run`, body e.g. `{"variables": {"$os": "Mac OS X"}}`.
3. Bind charts/tables to results; bind Retool inputs to body variables for live filtering. Retool AI can scaffold this from a plain-English prompt.

## Performance & cost tips

- Filter early (`WHERE`), use appropriate date ranges, select only needed columns (no `SELECT *`).
- Avoid scanning the same table multiple times; consider depending on a materialized view.
- Cache durations: 5–15 min (frequently changing), 1–6 h (periodic dashboards), 24 h (slow-changing/historical).
- Materialize high-volume expensive endpoints; deactivate unused endpoints/versions and disable materialization where it doesn't pay off (it consumes resources on a schedule).
- Pricing (not finalized; free for now) will bill on **compute (CPU time)** + **data scanned**.

## Troubleshooting

| Symptom | Cause / fix |
| --- | --- |
| 404 endpoint not found | Wrong name, inactive endpoint, or wrong project id |
| 401/403 | Bad personal API key, missing `endpoint:read` scope, or missing `Authorization` header |
| "Version X not found" | Check versions in UI, or omit `version` to use latest |
| "Variable 'X' is not allowed" | SQL endpoints accept only declared variables; insight endpoints accept breakdown name + `date_from`/`date_to` (non-materialized only) |
| "Query references undefined variable(s)" | Define the variable with `code_name` matching the `{variables.X}` placeholder and link it to the endpoint |
| "Cannot materialize endpoint" | Unsupported operator/placement, or inactive endpoint — see compatibility table |
| 429 too many concurrent requests | Hit concurrency limit; reduce parallel calls or materialize for higher limits |
| Column type error | Some column types unsupported; cast to supported types or simplify |
| Slow responses | Filter Usage tab to the endpoint; optimize query; materialize |

Breaking change: the `filters_override` parameter was removed — use `variables` instead.
