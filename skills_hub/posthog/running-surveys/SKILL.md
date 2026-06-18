---
name: running-surveys
description: Guides a product manager in using PostHog Surveys to collect qualitative user feedback and make product decisions. Covers creating surveys, question types (NPS, CSAT, rating, multiple choice, single choice, freeform text, link), display conditions and targeting (URL, device, feature flag, person/group properties, event-triggered), conditional/branching questions, repeating surveys, viewing and filtering results, building survey insights, summarizing responses, and sending responses to destinations like Slack and Zapier. Use when a PM asks how to measure NPS or CSAT, gather feature feedback, find out why users churned, validate an idea with a survey, run a fake-door test, read survey results, or pipe responses elsewhere.
---

# Running surveys in PostHog

PostHog Surveys collect **qualitative** feedback (the "why") to complement product analytics (the "what"). This skill helps a PM spec, target, launch, read, and act on surveys. It is light on SDK code — a PM specs surveys in the UI; developers wire up only API-mode or custom surveys.

> Self-hosted note: Surveys are a core product feature available on self-hosted instances (requires the surveys app enabled in the project and `posthog-js` 1.67.1+). The AI features (PostHog AI / Max survey creation and response summarization) and some managed data-pipeline destinations are cloud-oriented; availability may differ on self-hosted — see the AI/destination notes below.

## PM question → feature map

| PM question | Survey approach |
|---|---|
| How loyal/satisfied are users overall? | **NPS** survey: rating question 0–10 |
| How satisfied with a specific experience (support, checkout)? | **CSAT**: rating question (emoji or 1–5 number) |
| How would users feel if they could no longer use the product? | **Product-market fit (PMF)** survey: single/multiple choice |
| What do users think of a new feature? | **Feedback** survey: freeform text, targeted to flag/page |
| Why did users churn / cancel? | **Churn** survey: single choice ("why?") + freeform follow-up |
| Which feature should we build next? | **Multiple choice** survey on preferences |
| Will anyone want this idea before we build it? | **Fake-door test**: link/notification survey or feedback button (see [tutorials](#tutorials-and-related-docs)) |
| What are the themes across hundreds of responses? | **Summarize with PostHog AI** (see below) |
| How do I get notified when someone responds? | **Destinations** → Slack / Zapier / webhook |

## Question types

All available for popover and API modes. Set these in the survey wizard's **Steps** section.

| Type | Use it for |
|---|---|
| **Freeform text** | Open-ended "why" / suggestions |
| **Rating – number** | NPS (0–10), CSAT (1–5), satisfaction scores |
| **Rating – emoji** | Lightweight CSAT / quick sentiment |
| **Single choice** | Pick one reason (e.g. churn reason) — supports branching |
| **Multiple choice** | Feature preferences, multi-select reasons |
| **Link / notification** | Drive to a form, scheduling link, or fake-door page |

> Multiple questions per survey require a paid Surveys subscription. The description field supports HTML (e.g. `<img>`).

## Create a survey (workflow)

1. Go to the **Surveys** tab → **New survey**. Choose a template, generate with PostHog AI, or open the full editor.
2. **Presentation** — pick one:
   - **Popover** — PostHog's prebuilt UI, bottom corner. Easiest; no code.
   - **API** — you build the UI; PostHog handles display logic + capture. Advanced (see [custom surveys](reference/advanced-and-custom.md)).
   - **Feedback button** — prebuilt tab or your own button; good for always-on feedback.
   - **Hosted** — PostHog-hosted external link or iframe embed; responses anonymous unless you pass `distinct_id`. Good for emails, no-code sites.
3. **Steps** — add question(s), labels, choices, descriptions, button text, confirmation message; add [conditional logic](#conditional--branching-questions).
4. **Customization** (popover) — colors, position, placeholder, branding toggle, shuffle order, appearance **delay (seconds)**, auto-dismiss confirmation. (Hosted: colors + placeholders only.)
5. **Display conditions** — who sees it (see below).
6. **Completion conditions** — cap total responses; toggle **partial responses** (on by default); set repeat behavior.
7. **Save as draft**, review, then **Launch**. Popover shows immediately to matching users; API needs your code deployed first; hosted gives you a shareable link.

## Targeting / display conditions

A user must meet **ALL** conditions (in-app only; hosted surveys show to anyone with the link). Set in **Display conditions**:

- **Linked feature flag** — only users with a flag on (also the way to target a non-behavioral cohort, and to gate feedback to a rollout).
- **URL targeting** — contains / exact / regex match on `window.location.href`.
- **Device type** — Desktop, Mobile, Tablet, Console, Wearable (parsed from userAgent).
- **Selector matches** — show when an element like `#my-button` / `.my-button` exists/appears (great for "after an action").
- **Person & group properties** — e.g. `is_paying=true`; includes a percentage rollout. Requires identified users.
- **User sends events** — show to users who fired a specific event this session (event-triggered survey).
- **Wait period** — hide from users who saw any survey in the last X days.

By default each user sees a survey **once** (until dismiss/complete). See [repeating surveys](reference/advanced-and-custom.md#repeating-surveys) for event-repeat, scheduled repeat, and "every time display conditions are met."

## Conditional / branching questions

After any question: **"After this question, go to:"** → Next question / a specific question / Confirmation or End.

For **single choice** and **rating** questions you can branch on the answer:
- Single choice → route per selected option.
- Rating → routes by bucket: ratings split into **negative / neutral / positive** (e.g. 1–5 scale), so an NPS detractor can get a different follow-up than a promoter.

Classic NPS pattern: rating 0–10 → if **0–6 (detractors)** ask "what went wrong?", if **9–10 (promoters)** ask "what do you love?". To make a mid-list question the last one, explicitly set its next step to **Confirmation/End**.

> Interactive preview of branching isn't available yet — test by releasing the survey to yourself first.

## Reading & interpreting results

Two places to view results:

**1. Survey page → Results tab.** Shows users **seen**, **dismissed**, and **responses**, plus per-question charts.
- Rating & multiple-choice charts are clickable to **filter responses** by an answer.
- Rating/NPS trend charts have **Open as new insight** → save to a [dashboard].
- **Copy response key** on each question card gives `$survey_response_{question_id}` for custom insights/destinations.
- Filter by **date range**, **Filter by response**, property/person/cohort/HogQL filters, and **Show archived**.

**2. Build your own insights** from four events:

| Event | Fires when | PM reads it as |
|---|---|---|
| `survey shown` | survey displayed | reach / impressions |
| `survey sent` | a question answered | responses (the answer payload) |
| `survey dismissed` | user explicitly closes | active rejection |
| `survey abandoned` | partial-completion user leaves mid-survey | drop-off / friction |

Filter insights by `Survey ID` or `Survey Name`. Break down answers with the SQL function `getSurveyResponse(question_index, question_id, is_multiple_choice)` (0-based index). Use `survey shown` vs `survey sent` for **response rate**, and `survey abandoned` for **drop-off / which question loses people**.

> Partial responses: each answered question emits a `survey sent` event; all share one `$survey_submission_id`, `$survey_completed` flips to `true` only on full completion, and billing counts one submission per `$survey_submission_id` (not per event).

## Decisions surveys support

- **NPS / CSAT trend** → track sentiment over releases; an Open-as-insight NPS chart on a dashboard becomes a north-star sentiment metric.
- **Churn reasons (single choice)** → prioritize the top cancellation driver.
- **Feature-preference multiple choice** → rank the roadmap by demand.
- **Detractor freeform via branching** → specific fixes, not just a number.
- **Fake-door / link survey** → validate demand before committing eng time.

## Summarizing responses with PostHog AI

Instead of reading hundreds of responses, ask PostHog AI to extract **themes**, **sentiment** (positive/neutral/negative), **actionable insights**, and **segment comparisons** (e.g. free vs paid). For NPS it explains *why* promoters/passives/detractors scored as they did. Example prompts: "Summarize responses from my latest NPS survey", "What do detractors (NPS 0–6) complain about most?", "Compare feedback from free vs paid users."

PostHog AI can also **create** surveys from a description (picks question types, targeting, branching) — e.g. "Create an NPS survey for users who signed up in the last 30 days." Refine with follow-ups like "add a follow-up question for detractors."

> Self-hosted note: PostHog AI survey creation/summarization is in beta and consumes AI credits; it depends on hosted LLM features and availability may differ on self-hosted. For setup and capabilities see the **using-posthog-ai-assistant** skill and `/docs/posthog-ai/start-here`.

## Sending responses to destinations

To get notified or sync responses externally (Slack notifications, Zapier zaps, custom HTTP webhook):
1. **Data pipeline → Destinations** → search & **Create** (or HTTP Webhook for custom).
2. Under **Match event and actions**, select **survey sent**.
3. Useful properties: `$survey_name`, `$survey_questions` (array of `id`/`question`/`response`), `$survey_response_{response_key}` (per-question; get the key via **Copy response key**), `$survey_completed`, plus person properties.

> Self-hosted note: destinations run on the PostHog CDP / data pipeline; some managed destination integrations may need extra self-host configuration — confirm the destination is available on your instance.

## Tutorials and related docs

- Send responses to [Slack](/tutorials/slack-surveys) / [Zapier](/tutorials/zapier-surveys); [analyze with ChatGPT](/tutorials/analyze-surveys-with-chatgpt); [fake-door test](/tutorials/fake-door-test); [beta feedback](/tutorials/beta-feedback); [custom surveys](/tutorials/survey).
- Deep config (presentation modes, hosted/embedded surveys, URL pre-fill, repeating surveys, API/custom implementation, install): see **[reference/advanced-and-custom.md](reference/advanced-and-custom.md)**.

## Related skills

- **using-posthog-ai-assistant** — create and summarize surveys with PostHog AI / Max.
- **managing-feature-flags** — link surveys to a flag rollout for targeted feedback.
- **building-cohorts-and-segments** — target surveys to specific user segments.
- **analyzing-product-insights** / **building-dashboards** — turn survey events into trends and dashboards.
- **routing-events-with-the-cdp** — configure survey-response destinations.
