# Reliability and debugging: errors, traces, links to error tracking and replay

## Table of contents
- [Errors tab](#errors-tab)
- [Trace timeline (debugging a single bad answer)](#trace-timeline)
- [Link to error tracking](#link-to-error-tracking)
- [Link to session replay](#link-to-session-replay)

## Errors tab

**PM question it answers:** "Are generations failing, what are the failure modes, and which to fix first?"

Groups and aggregates error messages from your LLM app. When an event has `$ai_is_error = true`, PostHog **normalizes** the message by replacing dynamic values (IDs, timestamps, numbers) with placeholders so similar errors group together. Example: two "Request req_… failed at … with status 429" become one `"Request <ID> failed at <TIMESTAMP> with status <N>"`.

**Metrics per normalized error:** traces, generations, spans, embeddings, sessions, users (all unique counts affected), days_seen, first_seen, last_seen. Use these to prioritize by blast radius.

**Investigate:** click an error row to drill into Traces filtered to that error — see inputs/outputs and full context around the failure.

**Trends:** **Error trends** button = all errors over time by type (top 10); the trend icon on a row = that specific error over time across all event types. Use to spot spikes and verify a fix worked.

**Capture:** most integrations auto-capture errors on failed API calls. Manual capture sets `$ai_is_error: true` and `$ai_error: "<message>"`. Override grouping with `$ai_error_normalized` if you want custom buckets.

**Decision:** fix the error with the largest user/trace impact first; track the row's trend to confirm a deploy fixed it.

## Trace timeline

**PM question it answers:** "Why did this specific bad/expensive answer happen?"

Click a trace to open its tree-structured **timeline** of every generation and span, with profiling details. Use conversation display options (expand all / user only / collapse except output + last input) to scan fast. Tool calls show as tags. Use the **Summary** tab (see quality reference) for a fast AI summary instead of reading raw messages. From here you can add the trace to a **review queue** or open the failing generation in the **Playground** to reproduce and iterate.

## Link to error tracking

**PM value:** jump from an exception in Error Tracking to the full LLM trace that caused it, and track error rates by model/prompt-version/segment, set alerts on thresholds, and analyze failure patterns.

**How (spec to engineers):** capture the exception with the same `$ai_trace_id` as the LLM call (`posthog.captureException(error, { $ai_trace_id: traceId, ... })`). Works for both LLM API failures and downstream validation/processing errors. Then in the error tracking dashboard, click the LLM trace button to see the prompts/responses/metadata behind the error. See `tracking-errors`.

## Link to session replay

**PM value:** watch the real user session before/during/after an AI interaction to understand the experience, find the exact session where a problem occurred, and correlate slow AI responses with user impact.

**How (spec to engineers):** pass the frontend session ID to the backend and set `$session_id` on the LLM event. Two options:
1. **Automatic header injection (recommended):** configure `posthog-js` with `tracing_headers: ['api.your-app.com']`; it adds `X-POSTHOG-SESSION-ID` and `X-POSTHOG-DISTINCT-ID` headers to fetch/XHR calls. Backend reads the header into `posthogProperties.$session_id`.
2. **Manual:** read `posthog.getSessionId()` on the frontend and pass it in the request body, then set `$session_id` on the backend LLM call.

Then in AI Observability, click the session replay button on a generation/trace to watch the interaction in context. See `reviewing-session-replays`.
