---
name: using-posthog-ai-assistant
description: Guides a product manager in using PostHog AI (the in-app Max assistant) and the PostHog MCP server to get product answers in natural language instead of building insights manually. Use when a PM asks "what changed across my product this week", "why did conversion/signups drop", "find session replays where users got stuck", "build me a dashboard", "summarize my experiment / survey responses", "investigate a traffic spike", or wants the AI to write SQL/HogQL, create insights, feature flags, experiments, surveys, or alerts. Covers chat vs platform UI, modes, plan mode, research/deep-research mode, session summaries, context, slash commands, memory, AI credits, MCP for AI coding agents, and the *-ai / *-mcp helpers across products. Flags self-hosted limitations prominently.
---

# Using PostHog AI (Max) + MCP as a PM

PostHog AI is an in-app AI product analyst. A PM uses it to **get answers without manually building insights** — ask in plain English, and it queries data, creates insights/dashboards, writes SQL, finds and summarizes session replays, and explains results. The MCP server exposes the same powers to AI coding agents (Claude Code, Cursor, etc.) for engineers, but a PM mostly lives in the in-app chat.

> Self-hosted note: PostHog AI is **cloud-only**. The docs state plainly: "PostHog AI is only available in the cloud-hosted PostHog app... you cannot self-host PostHog AI or use your own models." On a self-hosted instance these chat/Max features and the AI-powered MCP tools are not available. The rest of this skill assumes PostHog Cloud (app.posthog.com). MCP non-AI tools (e.g. creating a flag) point at your account's region; AI-powered MCP tools still require cloud AI consent.

## Before you start: enable it (one-time, admin-gated)

1. Open the chat: click the PostHog AI / sparkles icon in the **right-hand sidebar** (or go to `#panel=max`).
2. The **first prompt** in an org triggers a consent prompt. An **org admin** must approve AI data analysis (Settings > Organization > General > "PostHog AI data analysis"). This is required because requests pass through third-party LLM providers (Anthropic, OpenAI, Alphabet; Azure for semantic search).
3. Run `/init` so it learns your product/company and saves it to memory ("onboard your robot product analyst").

It works best once you have data in at least one product (analytics, replays, flags, error tracking).

## PM question → what to ask PostHog AI

| PM question | Ask in chat | What it does / how to read it |
|---|---|---|
| "What changed across my product this week?" | `What changed across my product this week?` | Investigates multiple metrics and explains shifts |
| "Why did conversion / signups drop?" | `Why did conversion drop last week?` (use **Plan** or **Research** mode) | Multi-step root-cause investigation |
| "Build me a dashboard" | `Build a dashboard for onboarding retention` | Creates a dashboard with related insights |
| "Show me X over time / a funnel / retention" | `Show me a funnel from signup to first project created` | Picks insight type (trends/funnel/retention/paths/lifecycle/stickiness), applies events, filters, breakdowns |
| "Find replays where users got stuck" | `Find session replays where users got stuck` | Filters recordings; can then summarize them |
| "Summarize what users do in these recordings" | `Summarize these session recordings` | Categorizes by behavior, builds a report (see Session summaries) |
| "Summarize my experiment results" | `Summarize experiment results for my latest rollout` | Stats summary + can summarize replays per variant |
| "What are the themes in my survey responses?" | `Analyze responses from the latest NPS survey` | Extracts themes + sentiment |
| "Write/fix a SQL query" | `Write a SQL query to find the top 10 events by count` | Generates/edits/fixes HogQL in the SQL editor |
| "Set up a flag / A-B test" | `Create a multivariate flag for dark mode` / `Set up a 70/30 A/B test` | Creates flag/experiment (see Feature flags mode) |
| "Alert me when X drops" | `Alert me when daily signups drop below 100` | Creates an alert on a trend metric |
| "How do I use feature X in PostHog?" | `How do I set up a feature flag?` | Answers from docs — **free, no AI credits** |

Full prompt catalog by product: see [reference/example-prompts.md](reference/example-prompts.md).

## Where PostHog AI lives (the UIs)

- **In-app chat** — primary interface, always in the right sidebar (`#panel=max`). Has history, context, slash commands, memory, mode selector.
- **Platform UI** — inline sparkles entry points embedded in features (e.g. the sparkles next to session replay filters, "Summarize session replays" button in an Experiment). Triggers Max in-context.
- **Website chat** — on posthog.com when logged out, but limited to docs Q&A only.

## Modes (how the agent specializes)

The agent auto-switches to the right mode for your request, or you pick one in the **mode selector** above the chat. History and context carry over. Each mode unlocks specific actions:

| Mode | PM uses it for |
|---|---|
| **Product analytics** (default) | Insights (trends/funnel/retention), dashboards, alerts |
| **SQL** | Custom HogQL, exploring data warehouse sources |
| **Session replay** | Filter recordings, AI summaries of sessions |
| **Error tracking** | Find error patterns, frequency, impact on conversion |
| **AI Observability** | Search LLM traces (cost, latency, errors) if you ship LLM features |
| **Survey** | Create/edit surveys, analyze responses for themes/sentiment |
| **Feature flags** | Create flags, set rollout/targeting, A/B experiments, results summaries |

Two special workflow modes for hard questions:

- **Plan mode** (closed beta): for multi-step tasks. Agent asks clarifying questions, writes a **plan document for your approval**, and makes **no changes** to insights/dashboards/data until you approve. Use for "why did conversion drop?", comprehensive dashboards, cross-product investigations. Enter via the **Plan** selector or by asking "create a plan for…".
- **Research mode / Deep research** (closed beta): heaviest workflow, uses Claude Opus 4.5 with extended thinking. Two phases — (1) Planning: clarifies goals and writes a research plan notebook you approve; (2) Execution: decomposes into parallel tasks, queries across all modes, and synthesizes a **comprehensive report notebook**. Can take up to ~60 minutes. Use for root-cause, cross-metric, strategic, or full-journey audits. Only selectable from a **new conversation**.

Full mode tool lists and Plan vs Research vs chat comparison: see [reference/modes-and-workflows.md](reference/modes-and-workflows.md).

## Session replay summaries (high-value PM workflow)

Instead of watching recordings, ask Max to summarize them. It picks up the **filters you've applied** to the recordings, watches sessions, categorizes by behavior pattern, surfaces errors, and builds a report — with a live progress bar and ETA. You can drill in ("show me the specific recordings where users rage-clicked"). Also available inside **Experiments** via the "Summarize session replays" button, which categorizes recordings by variant to explain *why* variants behaved differently.

**Decision support:** confirm whether a metric drop is UX confusion vs a bug, and which exact sessions to show stakeholders.

## Investigating a traffic spike / drop (worked recipe)

Real PostHog-team workflow, chained one prompt at a time:
1. `Add a graph series for unique users.`
2. `There's an odd traffic spike around Sept 29. Create a dashboard with breakdowns by referral, country, user agent, and OS to find the root cause.`
3. `Create an insight with traffic breakdown by IP address. Any suspicious behavior?`
4. `Find the session replays for that IP address over the last 60 days.`
5. `Summarize these recordings in a report with key findings.`

This pattern (insight → breakdowns → drill to replays → summarize) generalizes to any anomaly investigation.

## Context, commands, and memory

**Context** — Max auto-detects your active tab/page. Add more with the **@ Add context** button: insights, events/actions, dashboards (with their insights), notebooks (full content). Context resets when you navigate to a different page, persists within a page; remove items manually. Adding context makes answers reference your real queries/filters.

**Slash commands:**

| Command | Does |
|---|---|
| `/init` | Collects product/company info → memory (run this first) |
| `/remember` | Saves your text to memory (10,000-char limit) |
| `/usage` | Shows current conversation's AI credit cost |

**Memory** — project-scoped, persists across conversations. Max remembers relevant details automatically; force it with "remember…" or `/remember`. Edit via the gear icon in chat or Settings > AI. Limit 10,000 chars; above 5,000 it only sees the first and last 2,500 (middle truncated), so **keep key facts at the start or end**.

## What it can read, and data governance (PM should know)

Reads (respecting your project [access controls](/docs/settings/access-control) — you only see what you can see): events, persons, sessions, groups, actions, cohorts, event/property definitions, data warehouse schema, saved insights and dashboards. It then writes queries, runs them, and summarizes — from a single number to a full user list.

Cannot (yet): access your source code or third-party tools, browse the web beyond PostHog docs, or read data outside the current project. Output is non-deterministic and not guaranteed accurate — verify before deciding.

To disable entirely: org admin toggles off "PostHog AI data analysis" in org settings (reversible; chat history is retained).

## AI credits (budget the PM should track)

AI features consume **AI credits** (100 credits = $1; ~20% markup over raw LLM cost). $5 / 500-credit free tier monthly. Complexity scales cost — a DAU question is cheap; summarizing hundreds of recordings is not. **Docs/help questions are free.** Beta features are free. Default $150 billing limit; set custom caps and usage alerts. Track with `/usage` in chat and the billing overview/usage pages (data may lag 24h). When credits run out, AI features pause until reset (prompt subscriptions pause; AI summaries in subscriptions are skipped but the rest still delivers).

## MCP and the *-ai / *-mcp helpers (point engineers here)

Across products PostHog ships paired helper docs: **`*-ai.mdx`** = use PostHog AI/Max in-app for that product (e.g. analyze-data-ai, find-replays-ai, analyze-experiments-ai, summarize-responses-ai, create-surveys-ai, manage-flags-ai, write-sql-ai, investigate-traffic-ai). **`*-mcp.mdx`** = do the same via the **MCP server** from an AI coding agent (build-insights-mcp, create-flags-mcp, create-experiments-mcp, run-sql-mcp, query-traffic-mcp, search-replays-mcp, debug-errors-mcp, query-revenue-mcp, etc.).

**PostHog MCP** is a free hosted endpoint (`https://mcp.posthog.com/mcp`) that lets engineers' AI agents read/write across PostHog from their editor — ship a flag from a prompt, triage a stack trace, run HogQL, set up a CDP destination, schedule reminders. PM value: your engineers can act on PostHog data without leaving code, and can run the same diagnose-a-conversion-drop / contain-a-regression recipes. Note: MCP **AI-powered** tools (e.g. natural-language→HogQL, sentiment/trace summarization) require the same org AI consent and bill as PostHog AI spend; plain CRUD tools are free.

Catalog of every *-ai/*-mcp helper and MCP use-case recipes: see [reference/mcp-and-helpers.md](reference/mcp-and-helpers.md).

Also relevant: the **AI wizard** (`npx -y @posthog/wizard@latest`) installs/audits PostHog in a codebase and can install the MCP server for an agent — an engineering task, not a PM one.

## PM workflow checklist

1. Confirm org admin has approved AI (Settings > Organization), else nothing works.
2. Run `/init`; `/remember` your product's north-star metric, key events, and naming.
3. For a quick number: just ask in default chat.
4. For "why did X happen?": switch to **Plan** mode (review the plan) or **Research** mode for a full report.
5. Add **@ context** (the relevant insight/dashboard) before asking, for sharper answers.
6. Name events explicitly and give a time range; iterate with follow-ups ("break this down by country").
7. Watch spend with `/usage`; remember docs questions are free.
8. Hand engineers the MCP + `*-mcp` helpers for in-editor work.

## Related skills

- using-posthog-product-analytics — building trends/funnels/retention/paths yourself
- using-posthog-session-replay — filtering and watching recordings
- using-posthog-experiments and using-posthog-feature-flags — A/B tests and rollouts
- using-posthog-surveys — survey creation and response analysis
- querying-posthog-with-sql — HogQL and the data warehouse
