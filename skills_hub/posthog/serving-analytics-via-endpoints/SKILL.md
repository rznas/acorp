---
name: serving-analytics-via-endpoints
description: >-
  Guides product managers on PostHog Endpoints — turning insights or SQL queries
  into stable, production API endpoints to expose analytics to your own
  customers, embed live metrics in your product, power landing pages, or feed
  internal tools (Retool). Use when a PM asks how to build customer-facing /
  embedded analytics, "let our customers see their own stats", expose PostHog
  data via API, show per-account metrics in-app, or pick Endpoints vs the Query
  API. Covers insight- vs SQL-based endpoints, variables, breakdowns, caching,
  materialization, data freshness, versioning, rate limits, usage analytics,
  OpenAPI SDK generation, security, and pricing trade-offs.
---

# Serving analytics via Endpoints

PostHog **Endpoints** turn an insight or SQL query into a stable, named API endpoint your app can call. This is the product surface a PM specs when the goal is **customer-facing / embedded analytics** — showing each customer their own stats inside your product, putting live metrics on a landing page, or feeding internal tools.

> Self-hosted note: Endpoints is in beta and free for now (pricing not yet finalized). Docs use `us.posthog.com` host examples; on self-hosted, substitute your own instance host (`<ph_app_host>`) in every API URL. Availability of beta features may differ on self-hosted — confirm Endpoints is enabled on your instance.

## When to reach for Endpoints (vs alternatives)

| PM question / situation | Use |
| --- | --- |
| Show analytics to our own customers in our app | **Endpoints** |
| Embed live metrics on a landing page or dashboard | **Endpoints** |
| High-traffic, production query that must be stable & fast | **Endpoints** |
| One-off, ad-hoc, exploratory query | [Query API](/docs/api/queries) |
| Bulk data export to S3 / Snowflake / a warehouse | [Batch exports](/docs/cdp/batch-exports) |

Why Endpoints over the raw Query API: isolated compute (no noisy-neighbor), built-in usage monitoring, versioning, OpenAPI specs, caching + materialization controls, and the ability to disable a misbehaving endpoint. PostHog discourages Query API for production and will eventually charge for it.

## The PM workflow (define → create → serve)

1. **Define the data.** Build an insight (Trends, Retention, Lifecycle) in Product analytics, OR write a query in the SQL editor. Verify it returns what you expect.
2. **Create the endpoint.** From an insight: three-dots menu → **Create endpoint**. From SQL: **Save as endpoint**. Give it a descriptive, stable name (`daily_active_users`, `account_users_activity`, `signup_funnel` — avoid `test`/`endpoint1`). The name is part of the URL, so treat renames as breaking.
3. **Verify in the Playground.** Open the endpoint → **Playground** tab → enter any variable values → **Execute**. Confirm the response shape before any engineering work starts.
4. **Serve it.** Engineers call `POST <ph_app_host>/api/projects/{project_id}/endpoints/{endpoint_name}/run`. For customer-facing use this MUST be proxied through your backend (see Security).
5. **Tune cost/speed** with caching / materialization, and **monitor** in the Usage tab.

## Two endpoint types — which to spec

| Type | Build it from | Variables | Best when |
| --- | --- | --- | --- |
| **Insight-based** | An existing Trends / Retention / Lifecycle insight | Automatic *magic variables* (breakdown properties, `date_from`/`date_to`) | You already have the insight; no SQL needed; simple trends/retention |
| **SQL-based** | A query in the SQL editor | Explicit `{variables.name}` you define | Custom aggregations, joins across tables, complex filtering, precise output, per-customer filters |

For per-customer analytics ("each customer sees only their data"), SQL-based with a required variable (e.g. `customer_id`) is the canonical pattern. See `reference/endpoint-config.md`.

## Variables — the key to per-customer / parameterized analytics

Variables customize results at execution time without creating many endpoints. **Required variables are a safety feature**: if an endpoint defines a variable, callers MUST pass it, which prevents accidentally returning unfiltered data to a customer.

- **SQL-based:** write `{variables.customer_id}` in the query, define the variable (name, type, optional default) in the Variables tab. Caller passes `{"variables": {"customer_id": "acme-corp"}}` in the POST body.
- **Insight-based (magic variables):** any breakdown property automatically becomes a variable (e.g. pass `{"$os": "Mac OS X"}` to filter a $os-breakdown insight). `date_from`/`date_to` filter the date range — but only on **non-materialized** insight endpoints.

Decision this supports: whether one parameterized endpoint can serve every customer (yes — filter by a `customer_id`/account variable) vs. needing separate endpoints. Full operator/materialization rules in `reference/endpoint-config.md`.

## Speed & cost: caching vs materialization

You set a single **data freshness** value (15 min → 7 days, default 1 day) and PostHog handles caching and materialization scheduling from it.

| | Caching (default) | Materialization |
| --- | --- | --- |
| What it does | Reuses recent query results within the freshness window | Pre-computes results on a schedule, stores in S3, serves those |
| Storage | Temporary | Persistent (S3) |
| Refresh | On demand | Scheduled (= data freshness) |
| Best for | Moderate traffic, fresher data | High traffic, expensive queries |
| Rate limits | Standard API limits | Much higher (see below) |

PM trade-off: shorter freshness = fresher data but more compute/cost; longer = faster & cheaper but staler. Most **customer-facing dashboards → daily**; near-real-time needs → hourly or shorter. Materialization saves money only for frequently-called, expensive queries; it can add cost for rarely-called or already-cheap ones.

> Materialization has limits: not all variable types/operators are supported, and cohort breakdowns or compare-mode queries can't be materialized. Details + variable-compatibility table in `reference/endpoint-config.md`.

## Rate limits (capacity planning for customer-facing scale)

- **Direct execution (non-materialized):** same as [standard API rate limits](/docs/api#rate-limiting).
- **Materialized endpoints:** burst **1,200 req/min**, sustained **12,000 req/hour**, **10 concurrent**.

Decision: if you're embedding analytics for many simultaneous customers, **materialize** to unlock the higher limits. Still not enough → contact PostHog.

## Versioning (ship query changes safely)

Every query edit auto-creates a new version (1, 2, 3…); old versions stay callable. Callers run the latest by default, or pin a version with `?version=2`. You can deactivate a single version (e.g. a buggy one) without disabling the whole endpoint. Tip: roll out a new version to a subset of users via [feature flags](/docs/feature-flags) before fully switching. PM value: change the underlying analytics without breaking customers already consuming v1.

## Usage analytics (monitor what you shipped)

The **Usage** tab shows total executions, bytes read, CPU time, avg & P95 duration, error rate, and the materialized-vs-direct split — with trend charts and a per-endpoint breakdown. Filter/break down by **endpoint**, **API key**, or **execution type**.

PM uses: see which customers/keys drive load, find slow or error-prone endpoints, and decide what to materialize or optimize.

## SDK generation for the consuming team

Every endpoint exposes an **OpenAPI 3.0 spec** at `.../endpoints/{endpoint_name}/openapi.json` (includes request variables, response schema, auth). Engineers generate a typed client in any language (e.g. hey-api for TypeScript, openapi-generator for Python). PM value: faster, type-safe integration; mention this when scoping the embed work.

## Security (must-have for customer-facing analytics)

1. **Never expose the PostHog API key client-side** — always proxy endpoint calls through your own backend.
2. **Use required variables** so an endpoint can't return unfiltered/all-customer data by accident.
3. **Authorize the request** — your backend must verify the logged-in user is allowed to query the requested `customer_id`/account before forwarding.

## AI & internal-tools helpers

- **Build with AI:** unsure what query to expose? Build the insight first with PostHog AI (see the `using-posthog-ai-assistant` skill). The troubleshooting page also has an AskAI input. > Self-hosted note: PostHog AI / Max may be cloud-only or need extra config on self-hosted.
- **Internal tools (Retool):** add a REST API resource pointing at `.../endpoints`, then call `/{endpoint_name}/run` with a `variables` body to power internal dashboards. See `reference/endpoint-config.md`.

## Quick checklist for a customer-facing analytics spec

- [ ] Pick type: insight-based (simple) vs SQL-based (per-customer / custom)
- [ ] Define a **required** filter variable (e.g. `customer_id`) so data is scoped per customer
- [ ] Verify shape in the Playground
- [ ] Choose data freshness; materialize if high-traffic/expensive
- [ ] Confirm backend proxy + per-user authorization (no client-side keys)
- [ ] Hand engineers the OpenAPI spec for a typed client
- [ ] Watch the Usage tab post-launch; version safely for changes

Deeper config (variable operators, materialization compatibility, timestamp bucketing, `refresh`/`debug` params, response format, troubleshooting, Retool steps): see `reference/endpoint-config.md`.

## Related skills

- `using-posthog-ai-assistant` — build the insight/query behind an endpoint with AI
- `building-product-analytics-insights` — create the Trends/Retention/Lifecycle insights that back insight-based endpoints
- `querying-data-with-sql` — author the SQL behind SQL-based endpoints
