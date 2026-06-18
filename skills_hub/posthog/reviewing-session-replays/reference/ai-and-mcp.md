# Replay AI and MCP helpers — reference

PM value of PostHog AI (Max) and the MCP server for session replay. See `using-posthog-ai-assistant` for the full assistant skill. See SKILL.md for the overview.

> Self-hosted note: these are hosted LLM features. Availability and setup may differ on self-hosted PostHog — confirm AI is configured for your instance before relying on these flows.

## Table of contents
- [Find replays with natural language](#find-replays-with-natural-language)
- [Summarize sessions](#summarize-sessions)
- [Search replays from your editor (MCP)](#search-replays-from-your-editor-mcp)

## Find replays with natural language

Describe the behavior instead of building filters by hand. PostHog AI understands replay-specific signals — rage clicks, dead clicks, error events, navigation patterns — plus events and any person property (plan, cohort, country). It translates your description into replay filters and returns matching sessions you can watch or summarize.

PM prompts:
- `Find sessions where users dropped off during checkout`
- `Find sessions with rage clicks on the pricing page`
- `Show me replays of users who signed up but never created a project`
- `Show recordings from enterprise users in the last 24 hours`

Tips: describe behavior not just pages ("got stuck during onboarding" > "/onboarding"); combine criteria; chain into a summary so you don't watch every replay.

## Summarize sessions

PostHog AI reads a recording's structured event stream (page views, clicks, inputs, scrolls, errors, custom events) — it does **not** watch the video — so summaries are fast even for long or bulk sessions. A summary covers key actions, behavioral patterns (loops, hesitation, rage clicks), errors encountered, and the session outcome (completed / dropped off / hit issues).

PM value: triage at scale. Instead of watching 20 recordings to spot a pattern, summarize all of them and surface what's common. Especially useful for **experiments** — metrics say which variant won, summaries say *why* (different navigation, confusion points, feature usage across control vs test).

PM prompts:
- `Summarize the last 5 session replays where users dropped off during checkout`
- `What patterns do you see in sessions with rage clicks?`
- `What are users struggling with on the pricing page?`
- `Summarize session replays for the control group in my checkout experiment`
- `What behavioral differences do you see between variant A and variant B?`

Tips: ask about specific behaviors; batch for patterns; combine with user properties; ask follow-ups like "what's the most common drop-off point?".

Requires PostHog AI set up and Session Replay enabled in the project.

## Search replays from your editor (MCP)

The PostHog MCP server lets an AI coding agent (Cursor, Claude Code, Codex, Windsurf, VS Code, etc.) search replays without opening the app — useful when a PM hands a bug to engineering. The agent can investigate bug reports, research current behavior before coding, validate after deploying, and search by user property. It filters by events, user properties, replay behaviors (rage/dead clicks, errors), and time ranges via the `query-run` tool.

Install via the wizard: `npx @posthog/wizard mcp add`.

Example prompts: `Find replays where users hit a JavaScript error on the /checkout page.` / `Are there any replays showing users stuck in a navigation loop?` / `Find the most recent session for user with email user@example.com.`
