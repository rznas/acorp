# Dashboard metrics: interpretation guide

Deep reading of the five customer analytics dashboard insights and what each tells a PM. Source: PostHog Customer Analytics docs (dashboard-metrics).

## Active users (DAU / WAU / MAU)

Unique users who performed your **activity event** daily, weekly, or monthly.

- This is the baseline pulse. If active users aren't growing, nothing else matters.
- Early on, do not obsess over the absolute number — watch the trend.
- Consistent week-over-week growth (even small) suggests you're building something people return to.
- Flat or declining active users is a clear signal the product isn't sticky enough yet.

## Power users

Users who performed the activity event the **most in the last 30 days** — significantly more than average.

- Power users are the canary in the coal mine. A small but growing group who use the product intensely means you likely have something worth scaling.
- They tell you what's working, become your word-of-mouth engine, and often pay first.
- PM action: identify who they are, understand *why* they're power users, and talk to them.

## Signups and signup conversion

New accounts created, and the % of signup-page viewers who complete signup.

- Signups measure whether positioning + acquisition attract the right people.
- Signup conversion specifically reveals friction in the signup flow.
- Caveat: signups alone are a vanity metric. A flood of signups with no activation/retention means wrong users or overpromising. Always analyze alongside engagement.

## Paying customers and free-to-paid conversion

Count of users on a paid plan, and % of free users who convert to paid.

- One of the strongest PMF signals — people paying validates the problem is worth solving.
- Free-to-paid conversion rate measures how effectively the product demonstrates value.

## Engagement (lifecycle)

Breakdown of customers by [lifecycle](/docs/product-analytics/lifecycle) stage relative to the activity event. Raw counts hide what's happening; the lifecycle composition is where the signal is.

Patterns to watch:

- **Healthy growth** — New users entering, returning users growing/stable, dormant low relative to active. This is what PMF looks like.
- **Leaky bucket** — Lots of new users, but dormant growing faster than returning. Acquiring but not retaining. Fix [activation](/newsletter/wtf-is-activation) and retention before scaling acquisition.
- **Resurrection spikes** — Sudden jump in resurrected users after a product change/campaign means something re-engaged churned users. Investigate the trigger.
- **Returning plateau** — Returning users flatten while new keep coming = stickiness ceiling. Core loop isn't compelling enough long-term.

Use lifecycle to stop celebrating vanity engagement and confront whether the base is sustainable and growing.
