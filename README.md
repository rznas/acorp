# acorp agents

Claude Code subagents. Each is a use-case agent that routes to a relevant subset of a larger skill library.

| Agent | Domain |
|-------|--------|
| `product-manager` | INSPIRED product discovery & validation, backed by PostHog data |
| `data-warehouse-architect` | Kimball dimensional modeling + Power BI semantic layer |
| `bi-report-builder` | Power BI reports/dashboards on dimensional models |
| `analytics-engineer` | End-to-end: PostHog instrument → Kimball model → Power BI visualize |

Install: copy `agents/*.md` into `.claude/agents/` (project) or `~/.claude/agents/` (global).
