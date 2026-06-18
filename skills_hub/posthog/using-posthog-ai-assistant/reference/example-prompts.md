# PostHog AI example prompts (reference)

Responses depend on your project's data, context, and prompts. Same prompt may produce different output (LLMs are non-deterministic). All prompts go in the in-app chat (`#panel=max`).

## Contents
- [Product analytics](#product-analytics)
- [Web analytics](#web-analytics)
- [Session replay](#session-replay)
- [Data warehouse / SQL](#data-warehouse--sql)
- [Feature flags](#feature-flags)
- [Experiments](#experiments)
- [Error tracking](#error-tracking)
- [Data pipelines](#data-pipelines)
- [AI Observability](#ai-observability)
- [Surveys](#surveys)
- [Docs / help (free, no AI credits)](#docs--help-free-no-ai-credits)

## Product analytics
- `Create an insight showing daily active users for the past week`
- `Build a dashboard for onboarding retention`
- `What changed across my product this week?`
- `Show me a funnel from sign up to first project created`
- `Which events have the highest drop-off rate in the past 7 days?`
- `Alert me when daily signups drop below 100`

PostHog AI builds every insight type: trends (counts over time), funnels (conversion between steps), retention (how often users return), paths (navigation), lifecycle (new/returning/dormant), stickiness (days users perform an action). Refine in place with follow-ups like "break this down by country" or "change to last 90 days". It can also create alerts on trend metrics and run correlation analysis.

## Web analytics
- `Create a dashboard showing web traffic by source and campaign`
- `Which landing pages have the highest bounce rate this week?`
- `Find the most common paths users take after visiting the blog`

## Session replay
- `Find sessions where users dropped off during checkout`
- `Summarize session replays where users got stuck`
- `Show recordings for users who triggered the rage click event`

It picks up the filters applied to recordings, categorizes by behavior pattern, surfaces errors, and builds a report with live progress + ETA.

## Data warehouse / SQL
- `List all available tables and their columns`
- `Write a query to join event data with user properties`
- `Fix the syntax for the query I have open in the SQL editor`
- `Last 10 Stripe charges` / `All charges last 14 days with email`
- `Make this query a CTE and bring in refunds as another CTE; union them`

## Feature flags
- `Create a new multivariate feature flag for dark mode`
- `Show me which flags are currently active in production`

## Experiments
- `Set up an A/B test with a 70/30 split for a new red button on the homepage`
- `Summarize experiment results for my latest feature rollout`

## Error tracking
- `Find the most common frontend errors this week`
- `Which errors impact user sign-ups the most?`
- `Summarize new issues introduced after the latest release`

## Data pipelines
- `Create a Hog function that filters out internal traffic`
- `Parse event timestamps into day, week, and month properties`
- `Drop events where test_mode = true or the environment is staging`

## AI Observability
- `Analyze LLM token usage over the past 7 days`
- `Summarize traces for the latest model deployment`

## Surveys
- `Create a new survey to collect feedback on pricing`
- `Analyze responses from the latest NPS survey`

## Docs / help (free, no AI credits)
- `Explain how feature flags work in PostHog`
- `Find setup instructions for Next.js SDK`
- `Summarize the differences between trends and funnels`

## Tips for better results
- Name events explicitly: `user_signed_up` beats "signups" if matching fails.
- Specify time ranges ("in the last 30 days", "this week").
- Describe the visualization ("as a funnel", "broken down by browser").
- Iterate with follow-ups to adjust filters/breakdowns/chart type.
- Ask questions ("Why did signups drop?"), not just commands — questions trigger investigation and explanation, not just a chart.
