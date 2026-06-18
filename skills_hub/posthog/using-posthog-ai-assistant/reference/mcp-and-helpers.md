# PostHog MCP server and the *-ai / *-mcp helper docs (reference)

## Contents
- [What MCP is and PM value](#what-mcp-is-and-pm-value)
- [Cost, billing, and AI consent](#cost-billing-and-ai-consent)
- [The *-ai vs *-mcp helper docs across products](#the--ai-vs--mcp-helper-docs-across-products)
- [MCP single-prompt use cases](#mcp-single-prompt-use-cases)
- [MCP multi-step recipes (PM-relevant)](#mcp-multi-step-recipes-pm-relevant)
- [Safety and scoping](#safety-and-scoping)
- [AI wizard](#ai-wizard)

## What MCP is and PM value

The PostHog **Model Context Protocol (MCP) server** is a free, hosted endpoint (`https://mcp.posthog.com/mcp`) that lets any MCP-compatible AI agent (Claude Code, Claude Desktop, Cursor, Codex, VS Code, Windsurf, Zed, PostHog Code) read and write across PostHog from plain-text prompts. Auth via OAuth (recommended) or a personal API key with the **MCP Server** preset; it auto-routes to your data region (US/EU).

PM value: this is the **engineer-facing twin** of in-app PostHog AI. Your engineers can ship a flag, triage a stack trace, run HogQL, build an insight, set up a CDP destination, or schedule a reminder without leaving their editor — using the same data Max uses. A PM rarely configures MCP directly but should know it exists so investigations and rollouts can move straight into code.

## Cost, billing, and AI consent

- Connecting and calling MCP tools is **free**; standard PostHog usage/billing applies to data your queries consume.
- Some MCP tools use **LLMs internally** (e.g. natural-language→HogQL `query-generate-hogql-from-question`, sentiment analysis, trace summarization). These are billed as **PostHog AI spend** and are only exposed when **AI data processing is enabled** in org settings (the same consent toggle that enables in-app PostHog AI). When AI is disabled, these tools are filtered out; plain CRUD tools remain.
- Subject to standard API rate limits (per team); AI-powered tools have lower additional limits.

> Self-hosted note: AI-powered MCP tools depend on cloud AI consent and providers, so they behave like the rest of PostHog AI (cloud-only). Plain (non-AI) MCP CRUD tools route to whatever account/region you authenticate with.

## The *-ai vs *-mcp helper docs across products

PostHog ships paired helper pages per product. `*-ai` = use **in-app PostHog AI / Max** for that product (PM-facing). `*-mcp` = do it via the **MCP server** from an AI coding agent (engineer-facing). Same underlying capability, different surface.

| Product | `*-ai` (in-app Max) | `*-mcp` (coding agent) |
|---|---|---|
| Product analytics | analyze-data-ai | build-insights-mcp |
| Web analytics | investigate-traffic-ai | query-traffic-mcp |
| Session replay | find-replays-ai, session-summaries-ai | search-replays-mcp |
| Experiments | analyze-experiments-ai | create-experiments-mcp |
| Feature flags | manage-flags-ai | create-flags-mcp |
| Data warehouse / SQL | write-sql-ai | run-sql-mcp |
| Error tracking | (covered in Max error mode) | debug-errors-mcp |
| Logs | search-logs-ai | debug-logs-mcp |
| Revenue analytics | analyze-revenue-ai | query-revenue-mcp |
| Surveys | create-surveys-ai, summarize-responses-ai | (via MCP survey tools) |
| AI Observability | analyze-traces-ai | query-traces-mcp |
| CDP / data pipelines | hog-functions-ai | (CDP destination tools) |
| Workflows | create-emails-ai | (workflow/email tools) |
| Customer analytics | configure-dashboard-with-ai | — |

When a PM needs a product answer, point to the `*-ai` page (in-app). When an engineer wants it in their editor, point to the `*-mcp` page.

## MCP single-prompt use cases

- "What are the top 5 errors in my project this week?" → ranked error groups + affected users
- "Show me the full stack trace for the most recent crash, then propose a fix."
- "Mark this issue as resolved and assign all future TypeErrors to the backend team."
- "How many unique users signed up in the last 7 days, broken down by day?" → trends query
- "What are the most common paths users take after signing up?" → paths query
- "Create a feature flag called new-checkout-flow enabled for 20% of users."
- "Create a multivariate flag with control 34 / variant_a 33 / variant_b 33." (integers summing to 100)
- "Create an A/B test for our pricing page that measures conversion to checkout."
- "Find users who triggered signup but didn't complete onboarding in the last 30 days." → HogQL
- "Create a Slack destination that sends a message when users sign up."
- "Create a dynamic cohort power-users for users who purchased >5 times in 30 days."
- "Remind me every Monday at 9am to review the activation dashboard." (linked reminder)

Tools are stable-named and can be scoped to a subset with the `tools` query parameter; sessions can be restricted to read-only.

## MCP multi-step recipes (PM-relevant)

**Diagnose a conversion drop:**
1. "Run a funnel from pageview on /pricing → clicked-checkout → completed-purchase over the last 14 days." (find the bleeding step)
2. "For users who dropped at the worst step, run a paths query showing where they went instead."
3. "Are there errors or slow logs correlated with that drop-off page in the same window?"
4. "Create an alert that pages me if pricing-to-purchase conversion drops >10% week-over-week."

**Launch a feature with full experimentation:**
1. "Create a feature flag new-onboarding-flow rolled out to 50%."
2. "Create an A/B test on it measuring the onboarding-completed event."
3. "Create an early access feature 'New onboarding' in beta stage for opt-in."
4. "Create a dashboard 'Onboarding launch' with experiment results, DAU, and the onboarding funnel."

**Investigate and contain a regression:**
1. "Show top errors from the last 24 hours, sorted by affected users."
2. "Pull the full stack trace and affected sessions for the top issue; explain root cause."
3. "Create a flag disable-broken-checkout at 100% and wire it into the failing path."
4. "Mark the issue resolved and add an assignment rule for similar TypeErrors."

## Safety and scoping

- Prompt injection is a real risk — review tool calls before executing.
- Restrict sessions to read-only tools, filter to specific tools/categories, and pin to a specific org/project to limit blast radius.
- The MCP server proxies to your instance and does not store analytics data; results return directly to the client.

## AI wizard

`npx -y @posthog/wizard@latest` — agentic CLI that installs/instruments PostHog in a codebase (SDKs, events, insights, dashboards) and can optionally install the MCP server for an agent. Also audits existing integrations (`wizard audit events|all|autocapture|feature-flags|identify|session-replay|web-analytics`), migrates from other vendors, wires up revenue analytics, and uploads source maps. This is an engineering task, not a PM one, but it's how engineers get the MCP server set up.
