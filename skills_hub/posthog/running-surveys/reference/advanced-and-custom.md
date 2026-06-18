# Surveys: advanced configuration & custom implementation

Deep detail behind the [running-surveys SKILL](../SKILL.md). PM-relevant items first; custom-code items at the end (developer territory — a PM specs them, doesn't write them).

## Table of contents

- [Presentation modes in depth](#presentation-modes-in-depth)
- [Hosted surveys](#hosted-surveys)
- [Embedding hosted surveys in iframes](#embedding-hosted-surveys-in-iframes)
- [Pre-filling responses via URL (one-click surveys)](#pre-filling-responses-via-url-one-click-surveys)
- [Repeating surveys](#repeating-surveys)
- [Completion conditions & partial responses](#completion-conditions--partial-responses)
- [Installation overview (PM awareness)](#installation-overview-pm-awareness)
- [API & custom surveys (advanced / developer)](#api--custom-surveys-advanced--developer)

## Presentation modes in depth

| Mode | UI built by | Display logic handled by PostHog | Best PM use |
|---|---|---|---|
| **Popover** | PostHog | Yes | Fast in-app feedback, NPS/CSAT, no code |
| **Feedback button** | PostHog tab or your button | Yes | Always-on "give feedback" |
| **API** | You | Yes (you fetch & render) | Custom in-app UI; needs dev work |
| **Hosted** | PostHog (external page) | n/a (link/iframe) | Email surveys, no-code sites, embeds |

Popover, API, and feedback-button surveys are **in-app** (rendered via SDKs and auto-linked to identified users). Hosted surveys are external URLs.

## Hosted surveys

- Select **Hosted Survey** as presentation type; on launch you get a shareable link (**Copy URL**).
- **No display conditions** apply — anyone with the link who hits a launched, still-collecting survey sees it.
- Responses are **anonymous by default**. To attribute to a user, append `?distinct_id=user123` using the *exact same* value passed to `posthog.identify()`.
- Add custom event properties via URL params, auto-captured on the survey-sent event: `?order_id=12345`. Types inferred as Numeric (matches `/^-?\d+(\.\d+)?$/`), Boolean (`"true"`/`"false"`), else String.
- Customization limited to colors and placeholders.

## Embedding hosted surveys in iframes

1. Create a **Hosted Survey**.
2. Enable **Allow embedding in iframes** in settings.
3. Save & launch.
4. On the overview page, **Copy embed code** → paste the HTML snippet anywhere (e.g. Framer, Webflow).

Notes: if the PostHog SDK is on the page, the default embed auto-attaches the person's distinct ID (`window.posthog.get_distinct_id()`); otherwise responses are anonymous. URL pre-fill works in embeds. For a custom iframe, use the hosted URL + `?embed=true`.

## Pre-filling responses via URL (one-click surveys)

Works for **single choice, multiple choice, and rating** questions (not open text). Great for one-click NPS in emails.

- `q{N}` = 0-based question index (`q0` first question); value = choice index or rating number.
- Single choice: `?distinct_id=user123&q0=2` (selects 3rd choice).
- Rating: `?distinct_id=user123&q0=9`.
- Multiple choice: repeat the param — `?q0=0&q0=2`.
- Multiple questions: `?q0=3&q1=8`.

Pre-filled questions set to **"Automatically submit on selection"** are skipped. With partial responses enabled, skipped answers record on page load. The survey starts at the first non-prefilled question. If everything is pre-filled, the user sees only the "Thanks!" confirmation.

> The `auto_submit` URL param and `autoSubmitIfComplete` / `autoSubmitDelay` SDK params are deprecated as of `posthog-js@1.302.0`.

**One-click NPS with optional follow-up:** Q1 = Rating with auto-submit on selection; Q2 = Freeform text; enable partial responses (Completion conditions → Response collection → **Any question**). Send `{survey_url}?q0=8`.

## Repeating surveys

Default: shown once per user until dismissed/completed, then never again. Exceptions:

- **Event-triggered:** for surveys triggered by an event, choose **Every time the event is sent** (repeats per event) or **Just once**.
- **Repeat on a schedule:** e.g. "Repeat 3 times, once every 30 days." Timer starts when the user completes *or* dismisses. Users must still match display conditions each time it becomes eligible. After the max repetitions, never shown again.
- **Every time the display conditions are met:** mainly for feedback-button surveys (shows each click). Combine with a **Wait time** (e.g. 30 days) to throttle.

## Completion conditions & partial responses

- **Stop after N responses** — rough cap; may slightly overshoot due to processing time.
- **Partial responses** (on by default; needs `posthog-js` 1.240.0+, web SDK only):
  - Disabled → one `survey sent` event on final question, `$survey_completed: true`.
  - Enabled → a `survey sent` event after the first answer (`$survey_completed: false`), again per additional answer (cumulative payload), and a final one with `$survey_completed: true`.
  - All events in one attempt share `$survey_submission_id`; **billing counts one per submission id**.
  - `survey abandoned` fires when a user answers ≥1 question then leaves before finishing (distinct from `survey dismissed`, which is closing without answering). Use it in insights to measure **drop-off**.

## Installation overview (PM awareness)

Popover/feedback/hosted surveys need only the surveys app enabled and PostHog installed. Quick install via the AI wizard command, or per-platform SDK guides (Web has the most complete feature support; mobile platforms have limitations — see `/docs/surveys/sdk-feature-support`). Web requires `posthog-js` 1.67.1+ with `opt_in_site_apps: true`.

```js-web
posthog.init("<ph_project_token>", {
    api_host: "<ph_client_api_host>",
    opt_in_site_apps: true,
})
```

## API & custom surveys (advanced / developer)

Use **API mode** when you want your own survey UI but still want PostHog to manage content, display conditions, and capture. A PM specs this; a developer implements it (JavaScript Web SDK only).

What PostHog still handles for you in API mode: survey content (editable at runtime without redeploy), and display conditions (dedupe, right survey, response caps, property/cohort/flag targeting).

Developer responsibilities:
- Fetch with `posthog.getActiveMatchingSurveys()` (or `posthog.getSurveys()`), filter `survey.type === 'api'`, and track "already seen" manually (the prebuilt app uses `localStorage`). Poll when using URL/selector targeting.
- Capture results so they appear in Results/insights:

```js-web
posthog.capture("survey sent", {
    $survey_id: "<survey-id>",            // required
    $survey_response: "<response>",        // required
    $survey_questions: [{ id: "<question-id>", question: "Do you like PostHog?" }], // required for getSurveyResponse
    $survey_name: "<survey-name>",         // optional
})
```

Also emit `survey shown` and `survey dismissed` analogously. A fully hardcoded survey (capturing `survey sent` from your own form) is possible but loses runtime content edits and automatic display logic — generally not recommended.
