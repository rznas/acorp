# Retention — detail

Shows how many users (or groups) return in subsequent periods. Source: `product-analytics/retention.mdx`.

## Questions it answers
- Are new signups coming back after trying the product?
- Is a new feature/UX getting people to return?
- What type of user retains best? (key PMF signal)

## Setup — two events
1. **Start event/action** — defines who is in a cohort.
2. **Return event/action** — proves they came back (often the same event).
- Aggregate by **Unique users** or by **group** (needs group analytics, e.g. retention by company/account).
- **Period**: any number of hours/days/weeks/months. For a full year use "last 12 months".
- **Cohorting**: **first time** (first time doing start event within the date range), **first ever occurrence** (very first instance across all time falls in the cohort period), **recurring** (did start event in that period regardless of whether first time).
- Filter by event/person properties, feature flags, or cohorts; add breakdowns.

## Reading the cohort table
- **Column A** = the cohort (a date range, one per period).
- **Column B** = cohort size.
- **Period 0** (Week/Day/Month 0) = always 100% (everyone did the start event).
- Later columns = % of that cohort who did the return event in that later period (e.g. 27.4% = 372 users).
- A user lives in only **one** start cohort, but can appear in multiple return periods — so a later period can exceed an earlier one (regular return cadence).
- Periods in progress are flagged and will update.
- Click any cell → list of retained users for deeper analysis.

## Calculation options
- **Retention reference**: **Period 0** (default, % vs initial cohort) vs **Previous period** (% vs prior period — surfaces the biggest period-over-period drop).
- **Returning time criteria**: **On** (returned in that exact period — for features needing return every interval) vs **On or after** (rolling/unbounded/cumulative — inverse of churn; for features that don't need every-period return).
- **Mean**: **Weighted** (default, larger cohorts weigh more) vs **Simple** (all cohorts equal).
- **Cohort label start index**: 0 (default) or 1, display only.

## Breakdowns
Break down by event/person/group properties, feature flags, cohorts, or SQL. Rule: **both** start and return event must share the same breakdown value (start in Chrome, return in Firefox → not counted in Chrome). Expand a row to see each cohort; dropdown filters the chart to one value. Example: break down by country to see which markets retain a feature best.

## Display
- Graph (line/bar) and/or cohort **table**. On a dashboard: Options → Show table only / graph only / both.

## Retention vs stickiness
Retention = % who came back **at least once** in a period (overall stickyness of the product). Stickiness = **how many** times they did it (engagement depth). Retention only tells you *if* they returned, not *how often* — use stickiness for frequency.

## Decisions
PMF assessment, whether onboarding/feature work lifts return rate, which segment retains best, where churn lives.
