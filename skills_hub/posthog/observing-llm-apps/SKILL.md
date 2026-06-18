---
name: observing-llm-apps
description: Guides product managers shipping LLM/AI features to use PostHog AI Observability (LLM analytics) to answer product questions about their AI product. Covers generations, traces, spans, sessions, embeddings, automatic cost calculation, sentiment classification, clusters, tools, errors, trace summarization, trace reviews, the playground, user feedback surveys, and links to error tracking and session replay. Use when a PM asks "what does our LLM feature cost", "cost per feature/user/model", "is our AI quality good", "are users frustrated with the chatbot", "why are generations failing", "what are users actually asking the AI", "which model should we pick", "how do I review bad AI responses", or wants an LLM cost/quality dashboard or to spec AI observability instrumentation.
---

# Observing LLM apps with PostHog AI Observability

For PMs whose team ships LLM features (chatbots, agents, RAG, copilots). PostHog AI Observability turns every LLM call into a queryable event so you can answer cost, quality, and reliability questions and decide what to fix, which model to use, and where to invest. You spec and read it; engineers wrap the SDK.

## How it works (one paragraph)

Engineers wrap your existing LLM provider SDK (OpenAI, Anthropic, Google, LangChain, Vercel AI, etc.). Calls still go to your provider; the wrapper sends a copy of the metadata to PostHog. Each LLM call becomes a **generation** (`$ai_generation` event) capturing model, provider, input/output, token counts, latency, and an **auto-calculated cost**. Generations roll up into **traces** (one request/conversation turn), which can roll up into **sessions** (a whole conversation/visit). **Spans** track non-LLM steps inside a trace (RAG retrieval, tool calls). Because these are normal PostHog events, you can build insights, dashboards, funnels, and SQL on them like any other event.

> Self-hosted note: Core capture, generations, traces, dashboard, cost calculation, tools, and errors are standard events and work on self-hosted. Features that run AI on your data — trace **summarization**, **clusters**, **sentiment** (local model), the **playground** trial models, and **PostHog AI / Max** natural-language analysis — depend on AI data-processing consent and PostHog-hosted/bundled models. Availability and setup may differ on self-hosted; verify these are enabled for your instance.

## PM question -> where to look

| PM question | Feature / tab | Read it as |
|---|---|---|
| What do our AI features cost in total / by model / by customer? | Dashboard (Costs), `$ai_total_cost_usd` insights | Spend trend; cost driver breakdown |
| Cost per conversation / per feature / per user? | Custom property on generations + insight | Tag traces with `feature`, group cost by it |
| Which features drive the most LLM spend? | PostHog AI prompt; insight grouped by feature property | Prioritize cost optimization |
| Are users using our LLM features? | Dashboard (Users, Generations) | Adoption of AI feature |
| Are users frustrated / happy? | [Sentiment](reference/quality-and-feedback.md) tab | Negative-sentiment traces to investigate |
| Do users rate responses good/bad? | [User feedback surveys](reference/quality-and-feedback.md) (thumbs up/down) | Quality signal tied to specific trace |
| What are users actually asking the AI? | [Clusters](reference/clusters-and-discovery.md) | Real usage patterns, unmet needs |
| Are generations failing? Why? | [Errors](reference/reliability-and-debugging.md) tab | Grouped failure modes, impact counts |
| Which tools/functions does our agent call? | [Tools](reference/clusters-and-discovery.md) tab | Tool usage, combinations, paths |
| Are responses slow? Latency spikes? | Dashboard (Latency), `$ai_latency` insights | p95/p99, slow calls |
| Which model should we pick? | [Playground](reference/playground-and-iteration.md) + cost/latency compare | Quality vs cost vs speed tradeoff |
| Why did this specific bad answer happen? | [Trace timeline](reference/reliability-and-debugging.md) + Summary tab | Step-by-step what the model saw/did |
| How do we QA AI quality repeatably? | [Trace Reviews](reference/quality-and-feedback.md) (in AI Evals) | Human scores on real traces |

## The data model PMs need to know

- **Generation** (`$ai_generation`): one LLM call. Has model, provider, `$ai_input`, `$ai_output_choices`, `$ai_input_tokens`, `$ai_output_tokens`, `$ai_latency`, `$ai_total_cost_usd`. This is the atom you build insights on.
- **Trace** (`$ai_trace_id`): required grouping = one full interaction (e.g. one chatbot message + the retrieval and tool steps behind it). The Traces tab lists them with person, total cost, total latency.
- **Span** (`$ai_span_id`): a step inside a trace that isn't an LLM call (RAG retrieval, function/tool call, validation).
- **Session** (`$ai_session_id`): optional, groups multiple traces (a whole conversation/thread/visit). You define what a session means.
- **Embedding**: a call to an embedding model (RAG indexing/search), tracked separately from generations.

Hierarchy: Session -> Trace -> (Spans and Generations). See [the data model reference](reference/data-model-and-events.md) for the full event-property catalog and the cost properties.

## What you can spec to engineers (high value, low effort)

PMs don't write SDK code, but these instrumentation asks unlock most PM questions. Pass them as `posthogProperties` / `posthog_properties` on the LLM call:

- **`feature`** (custom property): tag every call with which AI feature it is ("docs_chat", "dashboard_agent"). Unlocks cost/quality/adoption per feature.
- **`posthogDistinctId`** + **`posthogGroups`**: attribute LLM usage and cost to a user and to a company/account. Unlocks cost-per-customer.
- **`prompt_version`** / `prompt_id`: tag the prompt version so you can compare quality/cost before vs after a prompt change.
- **`$ai_trace_id`** consistent across a request, and **`$ai_session_id`** for a conversation, so traces and sessions group correctly.
- **`$session_id`** on backend LLM calls -> links to [session replay](reference/reliability-and-debugging.md) so you can watch the user's experience.
- **`$ai_trace_id`** on captured exceptions -> links to [error tracking](reference/reliability-and-debugging.md).

Engineers get cost, tokens, latency, model, tool calls, and errors automatically — no extra work.

## Cost: what's automatic and what to know

- Cost (`$ai_total_cost_usd`) is computed automatically from input/output (and cache, request, web-search, reasoning) tokens, matched against OpenRouter pricing with a fallback database.
- If you negotiated rates or use an unsupported model, engineers can set **custom per-token pricing** or send **pre-calculated costs**. See [data model reference](reference/data-model-and-events.md#cost-calculation).
- **Decision use:** group `$ai_total_cost_usd` by `feature`, `model`, or the company group to find your most expensive feature/customer and whether a cheaper model is viable.

## Workflow: investigate "is our AI any good / costing too much?"

1. Open the **AI Observability dashboard** for the cost, latency, generations, and users overview.
2. Break cost down by model and by your `feature` property to find the biggest spender.
3. Check the **Errors** tab for failure rate and grouped failure modes; click an error to see affected traces.
4. Check the **Sentiment** tab and **user feedback** (thumbs) for quality/frustration signal.
5. Open **Clusters** to see what users actually ask and which clusters have high cost/error rate.
6. Drill into representative **traces** (use the **Summary** tab for fast understanding); add bad ones to a **Trace Review** queue.
7. Test a fix or alternate model in the **Playground**, comparing models side by side on quality, latency, and tokens.
8. Decision: ship a prompt change, switch a model, add a guardrail, or fix a tool — then re-check the same metrics.

## Privacy & retention (PM should know)

- **Privacy mode** drops `$ai_input` and `$ai_output_choices` so prompts/completions are never stored — relevant for sensitive products.
- Large `$ai_` content properties (input, output, tools, states) are deleted after **30 days**; metadata (model, cost, tokens, latency, trace IDs) persists. To keep raw prompts/completions longer, set up a batch export. See [data model reference](reference/data-model-and-events.md#privacy-and-retention).

## AI helpers for PMs

- **PostHog AI (Max)**: on any AI Observability page, ask in natural language — "What are the most expensive LLM calls today?", "Which features drive the most LLM spend?", "Show traces with errors in the last 24h", "Compare costs between GPT-4 and Claude across my features." It queries your trace data and returns details. See `using-posthog-ai-assistant`.
- **MCP server** (`query-traces-mcp`): your coding agent (Cursor, Claude Code, etc.) can check LLM costs and errors from the editor via `get-llm-total-costs-for-project` and `query-run`. Useful for catching cost regressions around deploys.

> Self-hosted note: PostHog AI/Max and the on-data AI features may require additional configuration or may be unavailable on self-hosted; confirm with your instance admin.

## Reference files

- [data-model-and-events.md](reference/data-model-and-events.md) — events, key `$ai_` properties, cost calculation, sessions, embeddings, privacy/retention.
- [quality-and-feedback.md](reference/quality-and-feedback.md) — sentiment, user feedback surveys, trace summarization, trace reviews.
- [clusters-and-discovery.md](reference/clusters-and-discovery.md) — clusters, scatter plot, tools tab.
- [reliability-and-debugging.md](reference/reliability-and-debugging.md) — errors tab, trace timeline, link to error tracking and session replay.
- [playground-and-iteration.md](reference/playground-and-iteration.md) — playground, model comparison, saving to prompts/evals.

## Related skills

`using-posthog-ai-assistant` (PostHog AI/Max + MCP), `running-ai-evals` (LLM-as-judge evaluations & prompt management), `building-dashboards`, `analyzing-with-insights`, `tracking-errors`, `reviewing-session-replays`, `running-surveys`.
