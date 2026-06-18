# Funnels — detail

Funnels visualize a flow and show where friction/drop-off is. Source: `product-analytics/funnels.mdx`.

## What you learn
Where people get stuck, who the successful/unsuccessful users are, which steps have the highest friction & time to convert, the real paths users take, whether product changes improve conversion over time, and how seasonality affects conversion.

## Setup
1. New insight → **Funnel**.
2. **Add steps**, first event → success event. Start simple; avoid optional steps (they skew results).
3. **Combine events inline** in a step (logical OR) instead of a custom action.
4. **Step order**:
   - **Sequential** — B after A, any events allowed between.
   - **Strict order** — B immediately after A, nothing between.
   - **Any order** — steps in any sequence.
5. Steps are **actions to perform, not filters** — two `clicked button` steps require two clicks.
6. **Filter steps** by event/person/group properties, autocapture elements, or SQL; or a **global filter** at the bottom. Person properties require identified events + person profiles.
7. **Exclusion steps** — remove users who did event X between two steps (excluded from the whole funnel).
8. **First occurrence** per step:
   - **First-ever occurrence** — only the user's very first event of that type counts; if it doesn't match the step's filter/date range the user is dropped entirely.
   - **First occurrence matching filters** — first event of that type that also matches the step's filters; earlier non-matching ones ignored.

## Conversion rate
- **Overall conversion** — each step relative to step 1 (whole-funnel health).
- **Relative to previous step** — each step vs the previous one; **lowest relative conversion = biggest opportunity**.

## Graph types
- **Conversion steps** — drop-off + counts converting between steps.
- **Time to convert** — friction; long time = harder step. Histogram of time-to-convert available.
- **Historical trends** — conversion rate over time by entry date; shows if fixes/features helped. Enable **Hide incomplete periods** so users still inside the conversion window don't drag the line down; enable trend lines for noisy data.

## Breakdowns + attribution
Break down by event/person properties or a single cohort (only one cohort at a time). When a property isn't on every step's event, choose **attribution type**:
- **First touchpoint** — first value seen across steps (one value per person).
- **Last touchpoint** — last value seen (one value per person).
- **All steps** — person can appear under multiple values for the steps they did with each value.
- **Specific step** — value at that step only; excludes people without a value there; a person can have multiple values; useful to show a single URL in the pageview column. (Called "any step" in unordered funnels.)
- Path-cleaning rules normalize `$current_url`/`$pathname` breakdowns (e.g. `/user/123` → `/user/:id`).

## Act on a funnel
- Click a step's chart/column to list **completed** vs **dropped** people → save as cohort, talk to them, watch session replays.
- `...` on a step → **show user paths** leading to / between / after step / after-or-before drop-off → opens a paths insight.
- **Seasonality**: use historical-trends graph type over a seasonal range (weekends for B2B, Black Friday, etc.) before concluding a real change occurred.
- **Correlation analysis** section auto-highlights significant events/properties when you have no hypothesis (paid; needs sample size).

## Group-level funnels
Set **Aggregating by** to a group type to track how many companies/accounts move through each step (needs group analytics).

## Decisions
Which step to fix first, whether a fix worked, who to interview, whether a change is seasonal vs real.
