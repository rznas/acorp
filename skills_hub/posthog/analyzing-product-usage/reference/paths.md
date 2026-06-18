# User Paths — detail

Follow users' journeys and find the biggest drop-offs / discovery gaps. Source: `product-analytics/paths.mdx`.

## Questions it answers
Where users get confused/stuck, which parts of the app are actually used, why users don't discover features, where new users land from marketing.

## Reading the diagram
- Each step = a page/event; horizontal bars = a user navigating one node to the next; the number = distinct users through that step.
- Hover a node to highlight all upstream/downstream links, see step conversion rate and average time from the previous step.

## Step actions (`•••` on a node)
- **Set as path start** / **Set as path end** — focus where users go after / how they arrive at a step.
- **Exclude path item** — drop that event from the view.
- **View funnel** — open a funnel with this event as step 1 (great for analyzing a drop-off).
- Click a step's **number** → list of users → **Save as cohort** or **View recording** (session replay).

## Configuration
- **Event types** (pick ≥1, any combination): **Page views** (`$pageview`, by `Current URL`, continuous across domains), **Screen views** (`$screen`, by `$screen_name`), **Custom events** (by name), **SQL expression** (custom step logic, e.g. only Chrome pageviews).
- **Wildcard groups** — collapse pattern URLs into one step, e.g. `/product/*`; on custom events matches the name, e.g. `dashboard *`. (advanced — paid)
- **Path cleaning rules** — regex aliases for dynamic URL segments (`/user/123` → `/user/:id`); applied automatically from project settings unless you toggle off "Apply global path URL cleaning".
- **Exclusions** — remove events entirely.
- **Min/max people per path** — reduce density (e.g. min 100 users).
- **Number of steps** — longer journey vs cluttered view (default 5).

## Why path numbers ≠ funnel numbers (by design)
- Paths show only the **top 50 transitions**; less common paths excluded.
- Numbers count **transition frequency**, not unique users (one user doing Home→Pricing in 3 sessions = 3).
- **Path length limit** = step count (default 5); extra events truncated; start+end defined collapses the middle into "...".
- **30-minute session windows** split paths; funnels use a conversion window (default 14 days).
- Best practice: paths to **discover** patterns → funnels to **validate** a specific flow → session replays to investigate individuals.

## Limitation
Group types are **not supported** for user paths (or lifecycle).
