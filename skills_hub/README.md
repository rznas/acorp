# Skills Hub

Aggregated skill library — 72 skills across 4 groups, each derived from a source book/product docs. Use-case **agents** (in `.claude/agents/`) reference the relevant subset for their job.

| Group dir | Count | Source | Domain |
|-----------|-------|--------|--------|
| `product_management/` | 12 | INSPIRED (Marty Cagan) | Product discovery, vision/strategy, OKRs, teams, culture |
| `posthog/` | 18 | PostHog docs | Product/web/revenue analytics, experiments, feature flags, surveys, replays, errors |
| `power_bi/` | 23 | Power BI / Fabric docs | Semantic modeling, DAX, visualization, reports, RLS, ALM |
| `kimball/` | 19 | Kimball DW/BI Toolkit | Dimensional modeling, bus matrix, ETL, conformed dimensions, industry patterns |

## Agents (use-case → skill groups)

| Agent | Skill groups used | What it does |
|-------|-------------------|--------------|
| `product-manager` | product_management + posthog | Runs INSPIRED discovery & validation, backs decisions with PostHog data |
| `bi-report-builder` | power_bi + kimball + posthog | Builds reports/dashboards on dimensional models; picks visuals & writes DAX |
| `data-warehouse-architect` | kimball + power_bi (modeling) | Designs the warehouse/semantic layer: bus matrix, dimensional models, ETL |
| `analytics-engineer` | posthog + kimball + power_bi | End-to-end: instrument (PostHog) → model (Kimball) → visualize (Power BI) |

Skill names are unique within each group; some names repeat across `power_bi` and `kimball` (e.g. retail/financial/services patterns) — always reference skills by **group-qualified path** (`skills_hub/<group>/<skill>/SKILL.md`).
