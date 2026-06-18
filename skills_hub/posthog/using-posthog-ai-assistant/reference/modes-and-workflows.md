# PostHog AI modes and workflows (reference)

## Contents
- [How mode switching works](#how-mode-switching-works)
- [Mode-by-mode tools](#mode-by-mode-tools)
- [Default tools (all modes)](#default-tools-all-modes)
- [Plan mode](#plan-mode)
- [Research mode (deep research)](#research-mode-deep-research)
- [Plan vs Research vs regular chat](#plan-vs-research-vs-regular-chat)
- [Full tool catalog](#full-tool-catalog)

## How mode switching works

The agent picks the best mode for your request, or you set it manually via the **mode selector** above the chat input. Switching is **automatic** (e.g. asking for custom SQL in Product analytics mode jumps to SQL mode) or **manual**. On switch: conversation history is preserved, the new mode's tools become available, and context about your task is maintained.

## Mode-by-mode tools

| Mode | Best for | Mode-specific tools |
|---|---|---|
| Product analytics (default) | Trend/funnel/retention insights, dashboards, behavior patterns | `create_insight`, `create_dashboard` / `upsert_dashboard`, `upsert_alert` (alerts on trend insights) |
| SQL | Complex/custom HogQL, exploring data warehouse, ad-hoc analysis | `execute_sql` |
| Session replay | Find recordings, AI summaries of sessions, sessions with errors | `filter_session_recordings`, `summarize_sessions` |
| Error tracking | Error patterns, frequency, impact, stack traces, root cause | `search_error_tracking_issues` |
| AI Observability | LLM traces — cost, latency, errors, model usage | `search_llm_traces` |
| Survey | NPS/feedback/custom surveys, targeting, response analysis | `create_survey`, `edit_survey`, `analyze_survey_responses` |
| Feature flags | Flags with rollout/targeting, A/B experiments, results | `create_feature_flag`, `create_experiment`, `experiment_results_summary`, `experiment_session_replays_summary` |

## Default tools (all modes)

Available regardless of mode:

| Tool | Description |
|---|---|
| `read_taxonomy` | Read data schema: events, properties, sample values |
| `read_data` | Read data warehouse schema, insights, dashboards, billing info |
| `list_data` | Browse PostHog entities with pagination |
| `search` | Search insights, dashboards, cohorts, actions, experiments, flags, notebooks |
| `web_search` | Search the web for product/company/industry context |
| `switch_mode` | Switch to another specialized mode |

## Plan mode

Closed beta. Structured workflow for complex multi-step tasks — the agent plans before acting.

Steps:
1. **Clarification** — interactive form (time period? segments? success metrics?)
2. **Planning** — presents steps, data sources, expected outputs
3. **Review** — you Approve / Request changes / Cancel
4. **Execution** — only after approval does it create insights, run queries, produce outputs

Until approved it **cannot** create/modify insights or dashboards, or run anything that changes data/content. It can explore your schema, search existing content, ask questions, read docs, and create plan notebooks (initially transient/visible only in the conversation; click **Create notebook** to save to a permanent URL, after which updates auto-sync).

Enter via the **Plan** option in the selector, or ask "create a plan for analyzing our onboarding funnel".

## Research mode (deep research)

Closed beta. The most thorough workflow. Uses **Claude Opus 4.5 with extended thinking**. For questions needing systematic multi-source investigation, metric correlation, deep cause/effect reasoning, and synthesis into recommendations.

Phase 1 — Planning: clarifies objectives/scope/success, creates a research plan notebook (questions, data sources, analyses, deliverables), you review and approve.

Phase 2 — Execution: decomposes plan into atomic tasks, runs independent tasks in parallel, gathers data across all modes, iteratively refines drafts, synthesizes a final polished report notebook.

Capabilities: extended thinking, all analysis modes, parallel execution, notebook creation (draft + final), iterative refinement.

Start via **Research** in the selector — only from a **new conversation**.

Tips: be specific about goals ("Why are enterprise users churning within 30 days of signup?" beats "Why are users churning?"), define success criteria, set scope (time/segment/area), review the plan carefully.

Example projects:
- "Investigate why trial-to-paid dropped 15% last month. Focus on enterprise; compare converters vs non-converters."
- "Research how the new onboarding flow impacts 30-day retention. Compare completers vs drop-offs at each step."
- "Analyze error patterns in checkout. Which errors most impact conversion, and what behaviors lead to them?"

## Plan vs Research vs regular chat

| Aspect | Regular chat | Plan mode | Research mode |
|---|---|---|---|
| Best for | Quick questions, simple tasks | Complex multi-step analyses | Deep, multi-faceted investigations |
| Model | Claude Sonnet | (chat models) | Claude Opus 4.5 + extended thinking |
| Planning | Immediate | Plan created first | Required, plan approval before research |
| Approval | Not required | Required before execution | Required |
| Duration | Seconds–minutes | Varies | Up to ~60 minutes |
| Output | Direct answers, insights | Outputs from approved plan | Comprehensive report notebook |

## Full tool catalog

Tools PostHog AI can call (from the tools reference):

| App | Tool | Description |
|---|---|---|
| Platform | `navigate` | Navigate UI, tabs, pages |
| Platform | `read_data` | Read project data |
| Platform | `read_taxonomy` | Read event/property taxonomy |
| Platform | `read_data.billing_info` | Check billing data |
| Dashboards | `create_dashboard` | Create dashboards with insights |
| Dashboards | `edit_current_dashboard` | Add insights to current dashboard |
| Dashboards | `analyze_dashboard_refresh` | Summarize significant trends after refresh |
| Data pipelines | `create_hog_function_filters` | Pipeline function filters |
| Data pipelines | `create_hog_function_inputs` | Manage Hog function variables |
| Data pipelines | `create_hog_transformation_function` | Write Hog transformation code |
| Data warehouse | `read_data.datawarehouse_schema` | Read warehouse schema |
| Data warehouse | `fix_hogql_query` | Auto-fix SQL |
| Data warehouse | `generate_hogql_query` | Write/tweak SQL |
| Docs | `search.docs` | Search PostHog docs (free) |
| Error tracking | `filter_issues` | Filter error issues |
| Error tracking | `find_impactful_issue_event_list` | Find issues affecting conversion/activation |
| Experiments | `experiment_results_summary` | Summarize experiment results |
| Experiments | `create_experiment` | Create an experiment |
| Feature flags | `create_feature_flag` | Create a flag |
| AI Observability | `search_llm_traces` | Filter LLM traces by date/model/cost/latency/errors |
| Notebooks | `create_notebook` | Create notebooks (markdown, insights, replays) |
| Product analytics | `create_and_query_insight` | Edit the viewed insight |
| Product analytics | `search.insights` | Search existing insights |
| Session replay | `search_session_recordings` | Search recordings |
| Session replay | `session_summarization` | Summarize sessions |
| Surveys | `analyze_survey_responses` | Extract themes/insights from responses |
| Surveys | `create_survey` | Create surveys |
| Workflows | `create_message_template` | Create email templates (from scratch or a URL) |
