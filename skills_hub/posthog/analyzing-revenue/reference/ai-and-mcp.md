# Revenue with PostHog AI & MCP

Fastest way for a PM to get revenue answers without building charts or SQL by hand.

> Self-hosted note: PostHog AI / Max is a hosted LLM feature and may be cloud-only or require extra configuration on self-hosted; availability may differ on self-hosted. The MCP server runs in your AI client but queries your PostHog instance — confirm your instance/API access supports it.

## PostHog AI (Max) — in-app, natural language

Ask in plain English; it queries your connected revenue sources (Stripe or custom events) and can build dashboards.

What it can do: track MRR/ARR/total revenue and growth over time; break down by product, plan, region, or customer segment; compare periods (this quarter vs last); identify growth drivers, churn patterns, and revenue concentration risk.

Metrics it calculates: MRR, ARR, ARPU, churn (revenue lost to cancellations), expansion (growth from existing customers), LTV by segment.

Example prompts:
- "Show my revenue for the past year"
- "Create a dashboard that breaks down revenue by product"
- "Compare MRR growth this quarter vs last quarter"
- "Which customer segments drive the most revenue?"
- "Show revenue trends for enterprise vs self-serve plans"

Tips: name the source ("from Stripe" / "from our `purchase_completed` events"); state the exact metric (MRR vs total revenue vs ARPU); add the breakdown dimension ("by plan", "by country"); set the time range.

## MCP server — from your code editor

Brings the same revenue queries into Cursor, Claude Code, Windsurf, VS Code, Codex, etc.

Tools:
| Tool | Purpose |
| --- | --- |
| `query-run` | Run a HogQL query against revenue data (incl. Stripe); supports MRR, cohort analysis, custom breakdowns |
| `query-generate-hogql-from-question` | Turn a natural-language revenue question into a HogQL query for review |
| `insight-create-from-query` | Save a revenue query as an insight and add it to a dashboard |

Example prompts: "What's our current MRR?", "Show revenue broken down by plan for the past 6 months.", "Which customer segments have the highest ARPU?", "Query stripe_subscriptions to find all active enterprise subscriptions.", "Create a revenue dashboard with MRR trend, churn rate, and ARPU by plan."

Install (recommended): `npx @posthog/wizard mcp add`.

## When to use which

- Quick answer or auto dashboard while working in PostHog → PostHog AI / Max.
- You're in your editor / coding agent and want revenue context inline → MCP.
- Reproducible, exact, or complex revenue logic → managed views in SQL (see managed-views-and-sql.md).

See the `using-posthog-ai-assistant` skill for setup and broader AI usage.
