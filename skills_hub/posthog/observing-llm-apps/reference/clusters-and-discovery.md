# Discovery: clusters and tool usage

## Table of contents
- [Clusters](#clusters)
- [Tools tab](#tools-tab)

## Clusters

**PM question it answers:** "What are users actually asking our LLM to do, and which patterns are problematic?" — without reading every conversation.

Automatically groups similar traces or generations by content. Runs **automatically daily** on the past 7 days of data; AI generates a title and description per cluster and computes per-cluster metrics (avg cost, avg latency, avg tokens, error rate, total cost).

**Requirements:** traces/generations captured, and **AI data-processing consent** enabled (same as trace summarization).

> Self-hosted note: relies on AI data processing and embeddings on PostHog infrastructure; availability/config may differ on self-hosted.

**Why PMs use it:**
- Discover real usage patterns (what users ask for) — surface unmet needs and roadmap ideas.
- Find problem areas — clusters with high error rate, cost, or latency.
- Monitor trends — daily runs show how usage evolves.
- Surface outliers — items that fit no cluster (the dashed-border **Outliers** cluster) = unusual behavior.

**Where to read it (AI Observability > Clusters):**
- **Distribution bar** — proportional size of each cluster; hover for name/count, click to jump.
- **Scatter plot** — each dot is one trace/generation, color-coded by cluster; larger dots are centroids. **Axes are meaningless on purpose** — only **distance** matters: close dots = handled similarly, far/isolated dots = outliers. Click a dot → trace/generation detail; click a centroid → cluster detail; drag to zoom, double-click to reset.
- **Cluster cards** — AI title/description, size (count + %), and metrics (avg cost, latency, tokens, error rate, total cost). Expand to preview items.
- **Cluster detail page** — title/description/count, focused scatter plot, paginated list of items with expandable AI summaries (flow diagrams, bullets, notes); click an item for the full trace timeline.
- **Trace view > Clusters tab** — which cluster(s) a specific trace belongs to.

**Clustering jobs** (Jobs button): up to **five** per team. Each job configures:
- **Name** (e.g. "Production GPT-4o")
- **Analysis level** — cluster **traces** (whole conversations, high-level view) or **generations** (individual calls, model-interaction analysis)
- **Event filters** — scope by property (e.g. one model); empty = all
- **Enabled** toggle

Jobs run during the next scheduled cycle; the run selector shows the job name. Use the level toggle on the Clusters page to filter which runs display.

**Decision:** invest where high-volume clusters have bad metrics (high cost/error/latency); kill or de-prioritize features that no cluster represents real demand for; act on outliers.

## Tools tab

**PM question it answers:** "Which tools/functions does our agent call, how often, and in what combinations?"

Aggregates tool calls across all `$ai_generation` events. Tool names are auto-extracted at ingestion (`$ai_tools_called`, `$ai_tool_call_count`) across OpenAI (Chat Completions + Responses), Anthropic (tool_use blocks), OpenAI Agents SDK, Vercel AI SDK, and other normalized formats — no setup needed.

**Metrics per tool:** total_calls, traces, users, sessions, days_seen, **solo_pct** (% of calls where it was the only tool used), first_seen, last_seen.

**Visualizations (open as new insights):**
- Per tool: **Trend** (usage over time), **Combinations** (common tool combos containing it), **Paths** (tool-call flow sequences from it).
- Across all tools: **Tool trends** (line per tool, top 10), **Tool co-occurrence** (2D heatmap of pairwise co-occurrence).

Click a tool name to drill into the **Generations** tab filtered to that tool. Tool calls also show as tags in Traces and Generations (>5 collapse behind "+N more").

**Decision:** see which tools deliver value vs are unused; spot bad tool-call sequences or unexpected combinations; inform agent design.
