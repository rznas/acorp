# Quality signals: sentiment, user feedback, summarization, trace reviews

## Table of contents
- [Sentiment classification](#sentiment-classification)
- [User feedback surveys](#user-feedback-surveys)
- [Trace summarization](#trace-summarization)
- [Trace reviews](#trace-reviews)

## Sentiment classification

**PM question it answers:** "Are users frustrated/happy with our AI?" without reading every message.

Labels user messages in traces and generations as **positive / neutral / negative**. Computed **on-demand** when you open the Traces or Generations tab, using a local ONNX model (cardiffnlp/twitter-roberta-base-sentiment-latest) running on PostHog infrastructure — **no data sent to third parties**, and only user input (`$ai_input`) is processed, not LLM output. Results cached 24h. Currently **beta**.

**Where to read it:**
- **Sentiment tab** — card-based browser of user messages; filter Positive/Negative/Both, adjust an intensity (confidence) threshold, click **View trace** to deep-link to the message.
- **Traces table** — per-trace color bar (green/gray/red); hover for score and max positive/negative across messages.
- **Trace detail** — overall bar in header plus per-generation sentiment to see how mood shifts across the conversation.
- **Generations tab** — per-generation sentiment bar.

Sentiment is per-message; trace/generation sentiment is the average. **Decision:** filter negative-sentiment traces to find where prompts/agent behavior frustrate users, then investigate or add to a review queue.

> Self-hosted note: runs on PostHog infrastructure; availability may differ on self-hosted instances.

## User feedback surveys

**PM question it answers:** "Do users rate this specific response good or bad?"

Native integration with PostHog **Surveys** to attach thumbs up/down (and optional follow-up questions) to a specific trace.

**Setup (high level):**
1. On any top-level trace event, open the **Feedback** tab and create a survey (starts with a thumbs up/down question; optionally ask a follow-up on thumbs down).
2. In-app, responses are captured with the `$ai_trace_id` attached. React apps use the `useThumbSurvey` hook (survey must begin with a thumbs up/down question); non-React or complex cases use manual event capture.
3. Best practice: **one survey per logical group of traces** (e.g. a separate survey for "docs chatbot" vs "dashboard agent") so you can read aggregate feedback per feature.

**Where to read it:**
- Per-trace: the trace's **Feedback** tab shows the response (or "shown, no response").
- Aggregate: open the survey in **Surveys** for overall thumbs up/down ratings and a response table; click the AI Observability icon to jump to the related trace.
- All traces with feedback via SQL: filter traces where `$ai_trace_id IN (SELECT $ai_trace_id FROM events WHERE event = 'survey sent')`.

**Decision:** thumbs-down responses (with follow-up text) tell you exactly which responses to fix; combine with sentiment for a quality picture.

## Trace summarization

**PM question it answers:** "What happened in this complex multi-step trace?" without reading raw inputs/outputs.

In a trace or generation, click the **Summary** tab for an AI-generated **title**, **ASCII flow diagram**, **summary points**, and **interesting notes** (errors, unusual patterns). Two modes: **Minimal** (3-5 bullets, fast overview) and **Detailed** (5-10 points, full context). Summaries are cached; rate them with thumbs up/down.

**Requires AI data-analysis consent** (Settings → Organization → General → PostHog AI data analysis; org-wide). Rate limits: 50/min burst, 200/hour sustained, 500/day.

> Self-hosted note: requires AI data-processing to be enabled; availability/config may differ on self-hosted.

## Trace reviews

**PM question it answers:** "How do we QA AI quality with human judgment, repeatably?"

The manual review workflow inside **AI Evals**. Humans inspect real production traces, apply structured scores, add reasoning, and work through queues. Saved per trace (one active review per trace).

**Three parts (in AI Evals > Reviews):**
- **Queues** — collect traces to review later (support escalations, feedback-flagged traces, error-linked traces, prompt failures, new-feature traces, high-value customer conversations). Open a trace → **Add to queue**. Work through pending traces with prev/next navigation; saving a review removes it from the queue.
- **Reviews** — all saved reviews; search by trace ID/comment, filter by scorer, open the trace, see score tags, reviewer, comment, timestamp.
- **Scorers** — reusable, **versioned** review fields. Types: **Categorical** (single/multi-select labels like Resolved/Escalate/Hallucination), **Numeric** (with optional min/max/step), **Boolean** (custom labels). Old reviews keep the scorer version they were saved with.

**Review a trace:** open it (from Traces or a queue) → **Review** → pick relevant scorers → fill values → add reasoning → save. You can save a note-only review with no scores.

**Reviews vs evaluations:** reviews = human judgment for escalations, QA, nuanced/manual spot checks; evaluations = automated LLM-judge/code checks at volume. Use them together — reviews surface failure modes you then encode as automated evals. See `running-ai-evals`.
