# Data model, events, cost, sessions, privacy

## Table of contents
- [Event hierarchy](#event-hierarchy)
- [Generations](#generations)
- [Key $ai_ properties](#key-ai_-properties)
- [Traces](#traces)
- [Spans](#spans)
- [Sessions](#sessions)
- [Embeddings](#embeddings)
- [Cost calculation](#cost-calculation)
- [Custom properties PMs should request](#custom-properties-pms-should-request)
- [Privacy and retention](#privacy-and-retention)

## Event hierarchy

```
Session ($ai_session_id, optional)
└── Trace ($ai_trace_id, required)
    ├── Generation ($ai_generation)   # an LLM call
    └── Span ($ai_span_id)            # a non-LLM step; can nest more spans/generations
```

- A trace is the required top-level grouping for LLM events.
- A trace contains multiple spans and generations.
- A span can be the parent of other spans; a generation can be the child of a span or a trace.

## Generations

`$ai_generation` = one LLM call (e.g. prompt to Claude + response). Captures model, provider, input messages, output, token counts, latency, and cost. They are standard PostHog events, so you can build any insight, funnel, or SQL query on them. Found in **AI Observability > Generations**, with filtering/search and a conversation view plus raw JSON.

Tool calls in a generation are auto-extracted and shown as tags. Generations can be scored by [evaluations](https://posthog.com/docs/ai-evals) (LLM-as-judge) and classified for sentiment.

## Key $ai_ properties

| Property | Meaning |
|---|---|
| `$ai_model`, `$ai_provider` | Model and provider (used for cost lookup) |
| `$ai_input` | Input messages (large; dropped after 30 days; privacy-mode excluded) |
| `$ai_output_choices` | Model output (large; dropped after 30 days; privacy-mode excluded) |
| `$ai_input_tokens`, `$ai_output_tokens` | Token counts |
| `$ai_latency` | Response latency |
| `$ai_total_cost_usd` | Auto-calculated total cost (the main cost metric) |
| `$ai_trace_id` | Groups events into a trace (required) |
| `$ai_session_id` | Groups traces into a session (optional) |
| `$ai_span_id`, `$ai_span_name` | Span identity; `$ai_span_name` is the label shown in the trace tree |
| `$ai_tools_called`, `$ai_tool_call_count` | Tools called and how many |
| `$ai_is_error`, `$ai_error`, `$ai_error_normalized` | Error flag, message, normalized message |

## Traces

A trace = a full interaction (one chatbot turn and everything behind it). The **Traces** tab autocaptures person, total cost, total latency, etc. Click a trace for a **tree-structured timeline** of every generation and span, with conversation display options (expand all / user only / collapse except output + last input). Tool calls appear as tags. Each trace shows an overall **sentiment** bar and can be opened in the **Summary**, **Clusters**, **Feedback**, and **Review** tabs.

## Spans

Non-LLM units of work inside a trace: vector/RAG searches, tool/function calls, RAG pipeline steps (retrieval, reranking, context building), data processing/validation. Give PMs visibility into where latency and failures happen between LLM calls. Generations are technically a kind of span (with `$ai_span_id`); most spans are non-generation operations.

## Sessions

`$ai_session_id` groups multiple traces into a logical unit you define — a conversation thread, workflow, or visit. Use it to compute aggregate metrics (total cost, tokens, latency per session, session duration) and segment analytics. Different from PostHog's frontend `$session_id`; LLM events are usually captured server-side, so the AI session ID is typically managed by your backend. You can reuse `$session_id` for both if you pass it from frontend to backend.

## Embeddings

Calls to embedding models that turn text into vectors for semantic search, RAG indexing/search, similarity matching, and batch processing. Tracked separately from generations so you can monitor vector-generation cost and volume in RAG products.

## Cost calculation

`$ai_total_cost_usd` is computed from:
- **Token-based costs** — input (prompt) and output (completion) tokens.
- **Cache costs** — cached token reads/writes when applicable.
- **Request-based costs** — some models charge per request.
- **Web-search costs** — some models charge per web search.
- **Reasoning/thinking tokens** — counted for models that support them.

Pricing is matched via OpenRouter (primary) using `$ai_provider` + `$ai_model`, with a manually maintained fallback database. Cache token accounting differs by provider: Anthropic/Claude is **exclusive** (cache tokens separate from `$ai_input_tokens`); OpenAI and most others are **inclusive** (cache tokens included). Auto-detected; overridable with `$ai_cache_reporting_exclusive`.

**Override options (precedence high to low):**
1. Pre-calculated costs: `$ai_input_cost_usd`, `$ai_output_cost_usd`, `$ai_request_cost_usd`, `$ai_web_search_cost_usd` — used as-is.
2. Custom per-token pricing: `$ai_input_token_price`, `$ai_output_token_price` (required pair), plus optional `$ai_cache_read_token_price`, `$ai_cache_write_token_price`, `$ai_request_price`, `$ai_web_search_price`. Prices are **per token**, not per million.
3. Automatic model matching (OpenRouter then fallback).

Cost metadata for debugging: `$ai_model_cost_used`, `$ai_cost_model_source` (`openrouter`/`manual`/`custom`/`passthrough`), `$ai_cost_model_provider`.

## Custom properties PMs should request

Set via `posthogProperties` (JS) / `posthog_properties` (Python). High-value asks:
- `feature` — which AI feature ("docs_chat", "dashboard_agent"). Enables per-feature cost/quality/adoption.
- `posthogDistinctId`, `posthogGroups` — attribute to user and company/account for cost-per-customer.
- `subscription_tier` — cost/usage by plan.
- `prompt_id`, `prompt_version`, `experiment_variant` — compare quality/cost across prompt changes.
- `$ai_span_name` — readable labels in the trace tree.

Custom properties appear in the dashboard for filtering, insights, and segmented dashboards (e.g. average cost per conversation by subscription tier).

## Privacy and retention

- **Privacy mode** (`privacy_mode` SDK config or `posthog_privacy_mode` per request) excludes `$ai_input` and `$ai_output_choices` from capture — use for sensitive prompts/completions.
- **30-day retention** of large content properties: `$ai_input`, `$ai_output`, `$ai_output_choices`, `$ai_input_state`, `$ai_output_state`, `$ai_tools`. A full copy goes to `ai_events`; a trimmed copy (no large props) goes to `events`. After 30 days the `ai_events` copy is deleted — older traces still show metadata but no prompt/output content.
- Metadata (model, provider, tokens, cost, latency, trace IDs) persists like any product-analytics event, so historical cost/usage/performance analysis is unaffected.
- To keep raw prompts/completions beyond 30 days, set up a **batch export** to your own storage.
- To read large content properties in SQL, query the `posthog.ai_events` table (not `events`).
