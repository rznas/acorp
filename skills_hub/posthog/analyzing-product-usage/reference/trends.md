# Trends — detail

Trends plot data from people, events, and properties over time. Default insight type. Source: `product-analytics/trends/*`.

## What trends answer
- How key metrics change over time; long-term patterns / cycles.
- How a specific change (release) affected usage.
- Usage of different features side-by-side.
- How event *properties* vary via aggregation (sum, average, percentiles).

## Setup flow
1. **New insight** → trends is the default. Default event is `Pageview` if pageviews exist, else `Screen`, else `All events`.
2. Change the event/action via the dropdown next to the `A` series icon. Search across all events and actions.
3. **Aggregation**: defaults to Total count; switch to Unique users, DAU, etc. (see aggregations reference).
4. **Combine events inline**: select multiple events in one series → treated as one series via logical `OR`. Avoids creating a custom action.
5. **Property filters**: click the filter icon on a series → Add filter (e.g. `Device type = Desktop`). Duplicate the series (copy icon) and change the value to compare (Desktop vs Mobile).
6. **Add graph series** for more comparisons.
7. **Time range**: default last 7 days; change range and grouping (second/minute/hour/day/week/month).

## Reading nuances
- **Dotted line** = period still collecting data (e.g. current week when grouped by week). Don't read it as a real drop.
- **Smoothing**: 7-day or 28-day rolling average (available when grouped by day) to cut noise and reveal long-term patterns.
- **Hide weekend data** (Options) for B2B products with weekend dips.
- **Confidence intervals** (Options → Show confidence intervals; default 95%, configurable 90/95/99): shaded band around the line. Wider = less certain. Comparing two lines: non-overlapping bands ≈ likely statistically significant; overlapping = collect more data. Especially useful with sampling or small samples.
- Very long ranges on high-volume events hurt query performance — narrow the range if slow.

## Save / share / distribute
- Auto-named from events+filters; edit name (pencil) and add a Markdown description.
- **Add to dashboard** (an insight can live on many dashboards).
- **Add to notebook** for ad hoc analysis.
- **Export/embed**: `...` menu → Share or embed (live iframe) or Export → PNG. For huge exports use batch exports.
- **Subscribe**: email, email list, or Slack channel, on a schedule.

## Decisions trends support
- Is the metric growing/shrinking; did a release move it; which feature/segment is winning; is engagement (count per user) deepening.
